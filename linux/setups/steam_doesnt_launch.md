### Run steam and get like this

    Running Steam on manjarolinux 15.12 64-bit
    STEAM_RUNTIME is enabled automatically
    Installing breakpad exception handler for appid(steam)/version(0)
    libGL error: unable to load driver: i965_dri.so
    libGL error: driver pointer missing
    libGL error: failed to load driver: i965
    libGL error: unable to load driver: swrast_dri.so
    libGL error: failed to load driver: swrast

- Solution

    find ~/.steam/root/ \( -name "libgcc_s.so*" \
                        -o -name "libstdc++.so*" \
                        -o -name "libxcb.so*" \
                        -o -name "libgpg-error.so*" \) -print -delete

it works for me

- Refer to

[https://forum.manjaro.org/t/steam-doesnt-launch/623](https://forum.manjaro.org/t/steam-doesnt-launch/623)
