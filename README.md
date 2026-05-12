One script, in-place Ubuntu installer (replace ChromeOS) on Chromebook. 

**Only x86 devices are supported at this moment.**

No USB drive, no RW_LEGACY, no WP unlocking needed!

All you need is:
1. [Turn on Developer mode on your Chromebook.](https://www.chromium.org/chromium-os/developer-library/guides/device/developer-mode/)
2. Once ChromeOS started in developer mode, just connect to Wi-Fi. **No need** to login with Google Account!
3. Press [Ctrl+Alt+F2`(Refresh/Forward)`] enter VT2 console, login with `root` (should be no password).
4. Enter following commands:
    ````bash
    cd /tmp
    curl -LOf github.com/liyafe1997/crobuntu/raw/main/crobuntu
    bash crobuntu
    ````
5. Follow the script prompt to continue (Basically select Ubuntu version and desktop environment)
6. Wait it to be finished. After reboot, you will have Ubuntu!

Note: default username: `ubuntu`, password: `ubuntu`.

