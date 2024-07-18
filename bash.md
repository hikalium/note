# Bash hack

```
echo "tsura" | echo "obase=16;ibase=16;`xxd -p -c 1 | tr '[:lower:]' '[:upper:]' | grep -v '^00$' | sed 's/^0//' | tr '\n' '+'`0" | bc
```
