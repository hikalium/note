# imagemagick

```
sudo apt install imagemagick

# This will be blocked due to security policy
convert 250607.pdf 250607-%03d.png

# Use this instead:
gs -dSAFER -r300 -sDEVICE=pngalpha -o '250607-p%03d.png' 250607.pdf
```

# ref

- https://ginpen.com/2019/12/03/convert-pdf-to-png-jpeg-via-imagemagick/
