---
layout: default
date: 2014-10-24
title: "Extracting Text from PDF and HTML: An Online Data Pipeline"
tags: Python Data Pipeline
---

# Extracting Text from PDF and HTML: An Online Data Pipeline

In this article I describe a data pipeline that extracts text from online PDF
and HTML sources
and stores the extracted and compressed text as blobs in a SQL database for
further processing.

## Motivation

I consume online media through different apps on different devices (plain
browsers, Feedly, Twitter, etc.)
and e-mail myself links to interesting reads for future reference.

Gmail addresses are great for this purpose since an e-mail sent to *your-gmail-
address+STRING@gmail.com* still
ends up in the inbox of *your-gmail-address@gmail.com*.
Combining this with [Gmail
filters](https://support.google.com/mail/answer/6579?hl=en) allows you to set up
a rule so that all e-mails received on *your-gmail-address+STRING@gmail.com* are
archived and
assigned a label.
I use this scheme to send interesting links to myself on *my-
address+Links@gmail.com* and collect them under
the tag *Links* - I further set up an contact in my Gmail called *Links* that
points to this address.

To date I have collected a couple of hundred links to blog articles, scientific
papers and the like that
I feel compelled to do interesting text mining with - but first I need to
extract all of that text.

In this blog article I describe a pipeline that automatically accesses my Gmail
account and extracts the readable
text from all links I have sent myself over time.
Of course I am opening a can of worms here since these e-mails come in lots of
different
formats and point to different materials:

- some e-mails are just plain text with one or multiple links
- many e-mails come straight out of Feedly on my iPad which tends to send entire
blog articles with HTML formatting
- many e-mails contain direct links to PDF files which are generally not as easy
to parse as plain HTML

Here I will use a rule-based approach to distinguish between these different
formats.

## Import Python Libraries

These are the Python imports I will need throughout the pipeline:


    import httplib2
    
    from apiclient.discovery import build
    from oauth2client.client import flow_from_clientsecrets
    from oauth2client.file import Storage
    from oauth2client.tools import run
    
    import base64
    import re
    import requests
    from lxml import etree
    from StringIO import StringIO
    import itertools as it
    
    import urllib2
    from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter
    from pdfminer.converter import TextConverter
    from pdfminer.layout import LAParams
    from pdfminer.pdfpage import PDFPage
    from cStringIO import StringIO
    
    from collections import defaultdict
    
    from lxml.etree import fromstring
    
    import sqlite3
    from datetime import datetime
    import zlib

## Set Up Access to Gmail

Google offer an API for many of their services including Gmail.
Their free of charge quota for the Gmail API is very generous so that there
should be no concern of surpassing this allowance.

To access Gmail through Python follow the steps outlined
[here](https://developers.google.com/gmail/api/quickstart/quickstart-python)
(all steps assume that you are logged into your Gmail account in the browser
where you carry them out):

- create a new project in the Google Developer Console
- on the page of your new project, make certain to set a product name in the
APIs&auth/consent screen tab
- download the JSON from the APIs&auth/credentials tab and store it in a
location accessible from this Python script (I called mine *client_secret.json*)

Get the Google API Python package

    pip install google-api-python-client

and the following code
([source](https://developers.google.com/gmail/api/quickstart/quickstart-python))
will
connect to the Gmail API and expose an API resource object pointed to by
`gmail_service`.


    # Gmail API setup code from: https://developers.google.com/gmail/api/quickstart/quickstart-python
    
    # Path to the client_secret.json file downloaded from the Developer Console
    CLIENT_SECRET_FILE = 'client_secret.json'
    
    # Check https://developers.google.com/gmail/api/auth/scopes for all available scopes
    OAUTH_SCOPE = 'https://www.googleapis.com/auth/gmail.readonly'
    
    # Location of the credentials storage file
    STORAGE = Storage('gmail.storage')
    
    # Start the OAuth flow to retrieve credentials
    flow = flow_from_clientsecrets(CLIENT_SECRET_FILE, scope=OAUTH_SCOPE)
    http = httplib2.Http()
    
    # Try to retrieve credentials from storage or run the flow to generate them
    credentials = STORAGE.get()
    if credentials is None or credentials.invalid:
      credentials = run(flow, STORAGE, http=http)
    
    # Authorize the httplib2.Http object with our credentials
    http = credentials.authorize(http)
    
    # Build the Gmail service from discovery
    gmail_service = build('gmail', 'v1', http=http)

## Retrieve Bodies of Relevant E-Mail Messages

To fetch all e-mail bodies of interest, the Gmail API resource `gmail_service`
is the only object that needs to be queried.
The [Gmail API reference](https://developers.google.com/gmail/api/v1/reference/)
explains how to use the different
resource types (labels, message lists, messages) that will be used in the next
steps.

The Gmail label of interest is known to me as *Links* but internally Gmail
identifies all labels by a unique ID.
The following code identifies the label ID that corresponds to the label named
*Links*:


    labels = gmail_service.users().labels().list(userId='me').execute()['labels']  # 'me' is the user currently logged-in
    label_id = filter(lambda x: x['name'] == 'Links', labels)[0]['id']

I can now query `gmail_service` for all e-mail messages with the label *Links*
(as identified by `label_id`).
The API returns messages in pages of at most 100 messages so that it is
necessary to track a pointer to the
next page (`nextPageToken`) until all pages are consumed.


    def get_message_ids():
        """ Page through all messages in `label_id` """
        next_page = None
    
        while True:
            if next_page is not None:
                response = gmail_service.users().messages().list(userId='me', labelIds=[label_id], pageToken=next_page).execute()
            else:
                response = gmail_service.users().messages().list(userId='me', labelIds=[label_id]).execute()
    
            messages = response.get('messages')
            next_page = response.get('nextPageToken')
    
            for el in messages:
                yield el['id']
    
            if next_page is None:
                break

To extract message bodies it is necessary to distinguish between `text/plain`
and `MIME` e-mails (to my understanding the latter allows
embedded images and such).
When an e-mail is just plain text then the body is found directly in the
`payload`, however when the e-mail is MIME the body is the first of potentially
many `parts`.
See the [message API
reference](https://developers.google.com/gmail/api/v1/reference/users/messages)
for more detail.


    def message_bodies():
        for ctr, message_id in enumerate(get_message_ids()):
            message = gmail_service.users().messages().get(userId='me', id=message_id, format='full').execute()
        
            try:
                body = message['payload']['parts'][0]['body']['data']  # MIME
            except KeyError:
                body = message['payload']['body']['data']  # text/plain
            
            body = base64.b64decode(str(body), '-_')
            
            yield body

## Parse URLs of Material of Interest

I think of every e-mail with the *Links* label as a pointer to a resource on the
Internet (be it a link to a webpage, PDF, etc.).
To extract URLs from plain text I will make use of the following regular
expression formulated by
[John
Gruber](http://daringfireball.net/2010/07/improved_regex_for_matching_urls):


    pattern = (r'(?i)\b((?:[a-z][\w-]+:(?:/{1,3}|[a-z0-9%])|'
               r'www\d{0,3}[.]|[a-z0-9.\-]+[.][a-z]{2,4}/)'
               r'(?:[^\s()<>]+|\(([^\s()<>]+|(\([^\s()<>]+\)))*\))'
               r'+(?:\(([^\s()<>]+|(\([^\s()<>]+\)))*      \)|'
               r'[^\s`!()\[\]{};:\'\".,<>?\«\»\“\”\‘\’]))')

When I send myself a link through the Feedly iPad app, Feedly usually pastes the
entire blog article in the body.
Hence some Feedly-originating e-mail bodies will contain multiple URLs (e.g.
links as part of a blog article) but they mostly
provide a link to the original story as the first URL in the e-mail.

Other times I may type an e-mail in which I copy and paste multiple links to
interesting resources - in which
case I want to extract all URLs from the body and not just the first one.


    def is_feedly(body):
        return 'feedly.com' in body


    def urls():
        for body in message_bodies():
            matches = re.findall(pattern, body)
            if is_feedly(body):
                match = matches[0]
                yield match[0]  # Feedly e-mail: first URL is link to original story
            else:
                for match in matches:
                    yield match[0]

I noticed a number of URLs in `urls()` that did not match my expectation - such
as links to disqus comments and a publisher.
This is where a rule-based filter comes in:


    exclude = ['packtpub.com', 'disqus', '@', 'list-manage', 'utm_', 'ref=', 'campaign-archive']
    def urls_filtered():
        for url in urls():
            if not any([pattern in url.lower() for pattern in exclude]):
                yield url

I quite often send myself a link to the Hacker News message thread on an article
of interest.

To extract the URL to the actual article it is necessary to parse the HTML of
the Hacker News discussion
and locate the relevant URL in a `td` element with the attribute `class="title"`
- a rule that works for these particular Hacker News pages.


    def is_hn(url):
        return 'news.ycombinator.com' in url


    parser = etree.HTMLParser()
    def urls_hn_filtered():
        for url in urls_filtered():
            if is_hn(url) and (re.search(r'item\?id=', url) is None):
                continue  # do not keep HN links that do not point to an article
            elif is_hn(url):
                r = requests.get(url)
                if r.status_code != 200:
                    continue  # download of HN html failed, skip
                root = etree.parse(StringIO(r.text), parser).getroot()
                title = root.find(".//td[@class='title']")
            
                try:
                    a = [child for child in title.getchildren() if child.tag == 'a'][0]
                except AttributeError:
                    continue  # title is None
    
                story_url = a.get('href')
                yield story_url
            else:
                yield url

Poor memory gets the better of me quite often and I end up sending the same link
to myself multiple times.
To avoid downloading the same resource twice and skewing follow-on work with
duplicates I filter out duplicate URLs with the following code:


    def unique_urls():
        seen = defaultdict(bool)
        for url in urls_hn_filtered():
            key = hash(url)
            if seen[key]:
                continue
            else:
                seen[key] = True
                yield url

## Extract Text from PDF and HTML Documents

Now that I have a stream of unique URLs of interesting resources provided by
`unique_urls` the
next step is to download each of these resources and extract the text contained
in them.

To extract text I will here distinguish between HTML and PDF:
PDF text extraction is handled by `pdfminer` and I am making use of
[this code](http://stackoverflow.com/a/23840353), while
HTML text extraction is handled by `lxml` and [this code
snippet](http://stackoverflow.com/a/23929292).


    # pdfminer code from: http://stackoverflow.com/a/23840353
    
    def pdf_from_url_to_txt(url):
        rsrcmgr = PDFResourceManager()
        retstr = StringIO()
        codec = 'utf-8'
        laparams = LAParams()
        device = TextConverter(rsrcmgr, retstr, codec=codec, laparams=laparams)
        # Open the url provided as an argument to the function and read the content
        f = urllib2.urlopen(urllib2.Request(url)).read()
        # Cast to StringIO object
        fp = StringIO(f)
        interpreter = PDFPageInterpreter(rsrcmgr, device)
        password = ""
        maxpages = 0
        caching = True
        pagenos = set()
        for page in PDFPage.get_pages(fp,
                                      pagenos,
                                      maxpages=maxpages,
                                      password=password,
                                      caching=caching,
                                      check_extractable=True):
            interpreter.process_page(page)
        fp.close()
        device.close()
        string = retstr.getvalue()
        retstr.close()
        return string


    def resource_text():
        headers = {'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:33.0) Gecko/20100101 Firefox/33.0'}
        html_parser = etree.HTMLParser(recover=True, encoding='utf-8')
        
        for url in unique_urls():
            if url.endswith('.pdf'):
                try:
                    text = pdf_from_url_to_txt(url)
                except:
                    continue  # something went wrong, just skip ahead
                
                yield url, text
                
            else:
                try:
                    r = requests.get(url, headers=headers)
                except:
                    continue  # something went wrong with HTTP GET, just skip ahead
            
                if r.status_code != 200:
                    continue
                if not 'text/html' in r.headers.get('content-type', ''):
                    continue
                    
                # from: http://stackoverflow.com/a/23929292 and http://stackoverflow.com/a/15830619
                try:
                    document = fromstring(r.text.encode('utf-8'), html_parser)
                except:
                    continue  # error parsing document, just skip ahead
                
                yield url, '\n'.join(etree.XPath('//text()')(document))

## Consume Pipeline and Store Extracted Text

The generator exposed by `resource_text` returns a stream of text extracted from
both PDF and HTML documents.

All that remains now is to consume this generator and store the extracted text
together with the source URL for future reference.

Since this pipeline consists of a number of failure-prone steps (communicating
with APIs, extracting text, etc.) I will also
want to make certain to catch errors and deal with them appropriately:
In the generator `resource_text` I already included a number of places where
code execution just skips ahead to the next resource in case of failure -
however failure may still happen and it will be best to catch those errors right
here where the pipeline is consumed.

I will be lazy here and skip all resources that generate an error anywhere in
the pipeline.
Hence I will consume the generator `readable_text` until it throws the
`StopIteration` exception and skip all other exceptions.

I also make use of the `finally` keyword to make certain that the database
connection is closed even when the
pipeline throws an exception.

I further use `zlib` to compress extracted text before storing it in the
connected SQLite database as a blob.


    text_generator = resource_text()
    
    try:
        db = sqlite3.connect('gmail_extracted_text.db')
        db.execute('CREATE TABLE gmail (date text, url text, compression text, extracted blob)')
        db.commit()
        db.close()
    except sqlite3.OperationalError:
        pass  # table gmail already exists
    
    while True:
        try:
            db = sqlite3.connect('gmail_extracted_text.db')
            
            url, text = text_generator.next()
            
            now = datetime.now().__str__()
            if isinstance(text, unicode):
                text = zlib.compress(text.encode('utf-8'))  # to decompress: zlib.decompress(text).decode('utf-8')
            else:
                text = zlib.compress(text)
            db.execute('INSERT INTO gmail VALUES (?, ?, ?, ?)', (unicode(now), unicode(url), u'zlib', sqlite3.Binary(text)))
            db.commit()
        except StopIteration:
            break  # generator consumed, stop calling .next() on it
        except Exception, e:
            print e
            continue  # some other exception was thrown by the pipeline, just skip ahead
        finally:
            db.close()  # tidy up

In my case the above pipeline extracted text from 571 resources and the
resultant database is 7.9 MB in size.
