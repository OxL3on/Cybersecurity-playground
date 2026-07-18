## Finding Configuration Files

```
for l in $(echo ".conf .config .cnf")
do
    echo -e "\nFile extension: $l"
    find / -name *$l 2>/dev/null |
    grep -v "lib\|fonts\|share\|core"
done
```

## Searching Inside Configuration Files

```
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib")
do
    echo -e "\nFile: $i"
    grep "user\|password\|pass" $i 2>/dev/null |
    grep -v "#"
done
```

## Searching for Databases

```
for l in $(echo ".sql .db .*db .db*")
do
    find / -name *$l
done
```

## Searching for Notes

```
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

## Searching for Scripts

```
for l in $(echo ".py .pyc .pl .go .jar .c .sh")
do
    find /
done
```

## Searching Log Files
```
grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND=\|logs"
```




