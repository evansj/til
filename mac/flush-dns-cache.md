# Flush DNS Cache

When messing around with DNS entries it's sometimes useful to forcibly flush your DNS cache
so that your Mac picks up the changes sooner. To do this on El Capitan, run:

    sudo killall -HUP mDNSResponder

I have this aliased as `flushdns` to make it easier to remember.