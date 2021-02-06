
## Process listener:

```
#!/bin/bash

old_process=$(ps -eo command)

while true; do
	new_process=$(ps -eo command)
	diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -v -E "procmon|ps|command"
	old_process=$new_process
done
```