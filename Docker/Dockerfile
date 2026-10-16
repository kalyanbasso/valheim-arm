FROM tsxcloud/steamcmd-wine-ntsync:latest

RUN apt update
RUN apt install -y \
        libatomic1 \
        libpulse-dev \
        libpulse0

# SteamCMD (Oct 2026) broke the emulated setup on ARM64 in three ways:
#  1) its 32bit binary now imports __sched_cpualloc/__sched_cpufree, which are
#     only wrapped in box64 builds from Sep 2026 on (ptitSeb/box64#4381)
#     -> install a current box64 on top of the base image
#  2) its launcher now looks for a native "linuxarm64" steamcmd, which Valve
#     never publishes, and otherwise lets bash run the 32bit binary directly,
#     which fails with "Exec format error"
#     -> STEAM_PLATFORM=linux32 points it back to the emulated linux32 binary
#        and DEBUGGER=box64 inside the wrappers makes the launcher run it
#        through box64
#  3) as a 32bit client it now resolves the "windows 32" depot set for
#     +@sSteamCmdForcePlatformType windows and fails with
#     "Failed to install app '896660' (Missing configuration)"
#     -> the entrypoint passes +@sSteamCmdForcePlatformBitness 64
RUN if [ "$(dpkg --print-architecture)" = "arm64" ]; then \
        apt update && apt install -y box64 && \
        printf '%s\n' \
            '#!/bin/bash' \
            'export STEAM_PLATFORM=linux32' \
            'export DEBUGGER=box64' \
            'exec /root/steamcmd/steamcmd.sh "$@"' \
            > /usr/local/bin/steamcmd.sh && \
        cp /usr/local/bin/steamcmd.sh /usr/local/bin/steamcmd && \
        chmod +x /usr/local/bin/steamcmd /usr/local/bin/steamcmd.sh; \
    fi

ENV STEAM_PLATFORM=linux32

ENV ENABLE_PLUGINS="false"
ENV SERVER_NAME="Valheim_Server"
ENV SERVER_WORLD="tsx_world"
ENV SERVER_PASSWORD=
ENV SERVER_VISIBILITY=1
ENV SERVER_SAVE_INTERVAL=1800
ENV SERVER_BACKUPS=4
ENV SERVER_BACKUP_SHORT=7200
ENV SERVER_BACKUP_LONG=43200

WORKDIR /root

COPY server/ defaults/server/

COPY emulators.rc .

COPY --chmod=775 load_emulators_env.sh /load_emulators_env.sh

COPY --chmod=775 print_app_versions.sh /print_app_versions.sh

COPY --chmod=775 entrypoint.sh ./

ENTRYPOINT ["./entrypoint.sh"]
