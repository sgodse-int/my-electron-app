## GUI on WSL: Using WSLg

### Verify WSLg is Working

```bash
echo $DISPLAY
```

Should return `:0` or similar.

### Reset in Case of Errors:

```bash
unset DISPLAY
wsl --shutdown  # in PowerShell
# Then relaunch Ubuntu and try again
```

Test with:

```bash
sudo apt install x11-apps
xeyes
```

