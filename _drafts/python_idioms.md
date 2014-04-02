# Counting

    count = sum([1 for element in filter(lambda x: condition(x), all_elements)])

# Lists

## Unfurling

    elements = [x for sub_list in lists for x in sub_list]
    
I think of the singe-line double for loop as shorthand for

    for sub_list in lists:
        for x in sub_list:
            x
