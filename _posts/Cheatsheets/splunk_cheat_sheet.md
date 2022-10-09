# Splunk Cheatsheet

## Basics

- `index=*` - Show data from all indices
- `index="name_of_index"` - Show data from specific index

### Operators

```bash
"fish" AND  "chips" 
"fish" or  "chips"
```

### Sorting

- `* | sort ip, -url` - Sort results by ip value in ascending order and then by url value in descending order
- `* | reverse` - Reverse the order of a result set
- `* | head 20` - Return the first 20 results
- `* | tail 20` - Return the last 20 results (in reverse order)

[Splunk Official Wiki Cheatsheet](https://wiki.splunk.com/images/2/2b/Cheatsheet.pdf)
