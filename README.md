## How to reproduce stale build issue

Please run:
```bash
cd b && yarn bsb -make-world -w
```

And in the other shell run:
```bash
cd c && yarn bsb -make-world -w
```
