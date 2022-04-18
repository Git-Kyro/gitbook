## Exec bash

```python
import subprocess
arg = "ls -l"
def execBashCommand(arg):
    process =subprocess.Popen(arg, shell=True, stdout=subprocess.PIPE, universal_newlines=True)
    for line in process.stdout:
        sys.stout.Write(line)
        sys.stdout.flush()
```