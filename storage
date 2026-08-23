#!/usr/bin/env bash

# storage - Disk usage and storage information
# Version: 1.5.0

set -o pipefail

VERSION="1.5.0"

SYM_TOTAL="󰰥"
SYM_FREE="󰯻"
SYM_USED="󰰨"
SYM_USAGE=""
SYM_DISK=""
SYM_MOUNT="󰉋"
SYM_FS="󰋊"
SYM_DEVICE="󰓥"
SYM_INODE="󰎚"
SYM_READ="󰍛"
SYM_WRITE="󰍜"

BAR_START=""
BAR_MIDDLE=""
BAR_END=""

C_CYAN=$'\033[1;36m'
C_GREEN=$'\033[1;32m'
C_YELLOW=$'\033[1;33m'
C_RED=$'\033[1;31m'
C_BLUE=$'\033[1;34m'
RESET=$'\033[0m'

MOUNT="/"
UNIT="GB"

NO_COLOR=false
NO_SYMBOLS=false
QUIET=false
SHOW_ALL=false

SHOW_MOUNT=false
SHOW_FS=false
SHOW_DEVICE=false
SHOW_INODE=false
SHOW_LABEL=false
SHOW_UUID=false
SHOW_TYPE=false
SHOW_SIZE=false
SHOW_PARTITION=false

SHOW_READ=false
SHOW_WRITE=false
SHOW_IOPS=false
SHOW_LATENCY=false

SHOW_BAR=false
SHOW_LARGE=false

show_help() {
    cat <<EOF

storage - Disk usage and storage information
Version: $VERSION

Usage:
  storage [OPTIONS]

Disk:
  -d <path>       Select mount point
  -a              Show all physical disks

Information:
  -M              Mount point
  -F              Filesystem
  -D              Device
  -I              Inode information
  -L              Disk label
  -U              UUID
  -T              Filesystem type
  -S              Disk size
  -P              Partition information

Usage:
  -B              Show usage bar
  -PCT            Show usage percentage
  -LARGE          Show largest directories

I/O:
  -IO             Read + Write
  -IR             Read
  -IW             Write
  -IOPS           IOPS
  -IL             I/O latency
  -IOA            All I/O information

Units:
  -u <unit>       Select display unit

  Decimal:
    B
    KB
    MB
    GB
    TB
    PB
    EB

  Binary:
    KiB
    MiB
    GiB
    TiB
    PiB
    EiB

  -G              Same as -u GB
  -Gi             Same as -u GiB
  -R              Same as -u B

Output:
  -q              Quiet output
  -nc             No colours
  -nsfnf          No Nerd Font symbols

Other:
  -h, --help      Show this help
  -V, --version   Show version

Examples:
  storage
  storage -u MB
  storage -u GB
  storage -u TB
  storage -u GiB
  storage -MFDI
  storage -MFDI -IO
  storage -MFDI -IOA
  storage -MFDI -IO -B
  storage -MFDI -IO -a
  storage -a -u TB
  storage -nc -nsfnf

EOF
}

die() {
    printf '%sstorage:%s %s\n' "$C_RED" "$RESET" "$1" >&2
    exit 1
}

has_cmd() {
    command -v "$1" >/dev/null 2>&1
}

valid_unit() {
    case "$1" in
        B|KB|MB|GB|TB|PB|EB|KiB|MiB|GiB|TiB|PiB|EiB)
            return 0
            ;;
        *)
            return 1
            ;;
    esac
}

normalize_unit() {
    case "$1" in
        b|B) echo "B" ;;
        kb|KB|Kb|kB) echo "KB" ;;
        mb|MB|Mb|mB) echo "MB" ;;
        gb|GB|Gb|gB) echo "GB" ;;
        tb|TB|Tb|tB) echo "TB" ;;
        pb|PB|Pb|pB) echo "PB" ;;
        eb|EB|Eb|eB) echo "EB" ;;
        kib|KiB|KIB) echo "KiB" ;;
        mib|MiB|MIB) echo "MiB" ;;
        gib|GiB|GIB) echo "GiB" ;;
        tib|TiB|TIB) echo "TiB" ;;
        pib|PiB|PIB) echo "PiB" ;;
        eib|EiB|EIB) echo "EiB" ;;
        *) return 1 ;;
    esac
}

while [[ $# -gt 0 ]]; do
    case "$1" in
        -h|--help)
            show_help
            exit 0
            ;;

        -V|--version)
            printf 'storage %s\n' "$VERSION"
            exit 0
            ;;

        -d)
            [[ -n "${2:-}" ]] || die "-d requires a path"
            MOUNT="$2"
            shift 2
            ;;

        -u|--unit)
            [[ -n "${2:-}" ]] || die "-u requires a unit"
            UNIT=$(normalize_unit "$2") ||
                die "invalid unit: $2"
            shift 2
            ;;

        -G)
            UNIT="GB"
            shift
            ;;

        -Gi)
            UNIT="GiB"
            shift
            ;;

        -R)
            UNIT="B"
            shift
            ;;

        -a|--all)
            SHOW_ALL=true
            shift
            ;;

        -nc|--no-colours|--no-colors)
            NO_COLOR=true
            shift
            ;;

        -nsfnf|--no-symbols)
            NO_SYMBOLS=true
            shift
            ;;

        -q|--quiet)
            QUIET=true
            shift
            ;;

        -B|--bar)
            SHOW_BAR=true
            shift
            ;;

        -PCT|--percentage)
            shift
            ;;

        -LARGE|--large)
            SHOW_LARGE=true
            shift
            ;;

        -IO)
            SHOW_READ=true
            SHOW_WRITE=true
            shift
            ;;

        -IR)
            SHOW_READ=true
            shift
            ;;

        -IW)
            SHOW_WRITE=true
            shift
            ;;

        -IOPS)
            SHOW_IOPS=true
            shift
            ;;

        -IL)
            SHOW_LATENCY=true
            shift
            ;;

        -IOA)
            SHOW_READ=true
            SHOW_WRITE=true
            SHOW_IOPS=true
            SHOW_LATENCY=true
            shift
            ;;

        -*)
            combined="${1#-}"

            [[ ${#combined} -ge 2 ]] ||
                die "unknown option: $1"

            for ((i=0; i<${#combined}; i++)); do
                char="${combined:i:1}"

                case "$char" in
                    M) SHOW_MOUNT=true ;;
                    F) SHOW_FS=true ;;
                    D) SHOW_DEVICE=true ;;
                    I) SHOW_INODE=true ;;
                    L) SHOW_LABEL=true ;;
                    U) SHOW_UUID=true ;;
                    T) SHOW_TYPE=true ;;
                    S) SHOW_SIZE=true ;;
                    P) SHOW_PARTITION=true ;;
                    B) SHOW_BAR=true ;;
                    *)
                        die "unknown option: $1"
                        ;;
                esac
            done

            shift
            ;;

        *)
            die "unexpected argument: $1"
            ;;
    esac
done

if $NO_COLOR; then
    C_CYAN=""
    C_GREEN=""
    C_YELLOW=""
    C_RED=""
    C_BLUE=""
    RESET=""
fi

if $NO_SYMBOLS; then
    SYM_TOTAL=""
    SYM_FREE=""
    SYM_USED=""
    SYM_USAGE=""
    SYM_DISK=""
    SYM_MOUNT=""
    SYM_FS=""
    SYM_DEVICE=""
    SYM_INODE=""
    SYM_READ=""
    SYM_WRITE=""

    BAR_START=""
    BAR_MIDDLE=""
    BAR_END=""
fi

unit_divisor() {
    case "$1" in
        B)   echo "1" ;;
        KB)  echo "1000" ;;
        MB)  echo "1000000" ;;
        GB)  echo "1000000000" ;;
        TB)  echo "1000000000000" ;;
        PB)  echo "1000000000000000" ;;
        EB)  echo "1000000000000000000" ;;
        KiB) echo "1024" ;;
        MiB) echo "1048576" ;;
        GiB) echo "1073741824" ;;
        TiB) echo "1099511627776" ;;
        PiB) echo "1125899906842624" ;;
        EiB) echo "1152921504606846976" ;;
        *) return 1 ;;
    esac
}

human_size() {
    local bytes="$1"
    local divisor

    divisor=$(unit_divisor "$UNIT") || return 1

    awk -v bytes="$bytes" \
        -v divisor="$divisor" \
        -v unit="$UNIT" \
        'BEGIN {
            printf "%.2f %s", bytes / divisor, unit
        }'
}

read_fs() {
    local target="$1"
    local line

    line=$(df -P -B1 "$target" 2>/dev/null | tail -1)

    [[ -n "$line" ]] || return 1

    FS_DEVICE=$(awk '{print $1}' <<< "$line")
    FS_TOTAL=$(awk '{print $2}' <<< "$line")
    FS_USED=$(awk '{print $3}' <<< "$line")
    FS_FREE=$(awk '{print $4}' <<< "$line")
    FS_PERCENT=$(awk '{print $5}' <<< "$line" | tr -d '%')

    [[ "$FS_TOTAL" =~ ^[0-9]+$ ]] || return 1
    [[ "$FS_USED" =~ ^[0-9]+$ ]] || return 1
    [[ "$FS_FREE" =~ ^[0-9]+$ ]] || return 1

    FS_TOTAL_H=$(human_size "$FS_TOTAL")
    FS_USED_H=$(human_size "$FS_USED")
    FS_FREE_H=$(human_size "$FS_FREE")

    return 0
}

parent_device() {
    local device="$1"

    [[ -n "$device" ]] || return 1

    if ! has_cmd lsblk; then
        printf '%s\n' "$device"
        return 0
    fi

    local pk

    pk=$(lsblk -no PKNAME "$device" 2>/dev/null | head -1)

    if [[ -n "$pk" ]]; then
        printf '/dev/%s\n' "$pk"
    else
        printf '%s\n' "$device"
    fi
}

calculate_width() {
    local max=0
    local value
    local length

    for value in "$FS_TOTAL_H" "$FS_FREE_H" "$FS_USED_H"; do
        length=${#value}

        if (( length > max )); then
            max=$length
        fi
    done

    if (( max < 10 )); then
        max=10
    fi

    VALUE_WIDTH=$((max + 5))
}

print_table() {
    local W=16

    printf '%b%s Total%b%*s' \
        "$C_CYAN" "$SYM_TOTAL" "$RESET" 8 ""

    printf '%b%s Free%b%*s' \
        "$C_CYAN" "$SYM_FREE" "$RESET" 9 ""

    printf '%b%s Used%b%*s' \
        "$C_CYAN" "$SYM_USED" "$RESET" 9 ""

    printf '%b%s Usage%b\n' \
        "$C_CYAN" "$SYM_USAGE" "$RESET"

    printf '%-*s' "$W" "$FS_TOTAL_H"
    printf '%-*s' "$W" "$FS_FREE_H"
    printf '%-*s' "$W" "$FS_USED_H"
    printf '%s%%\n' "$FS_PERCENT"
}

print_bar() {
    local BAR_WIDTH=40
    local percentage="$FS_PERCENT"
    local filled

    if ! [[ "$percentage" =~ ^[0-9]+([.][0-9]+)?$ ]]; then
        percentage=0
    fi

    (( percentage < 0 )) && percentage=0
    (( percentage > 100 )) && percentage=100

    filled=$(awk \
        -v p="$percentage" \
        -v w="$BAR_WIDTH" \
        'BEGIN {
            printf "%d", (p * w / 100) + 0.5
        }'
    )

    printf '\n'
    printf '%b%s Usage%b\n' \
        "$C_CYAN" "$SYM_USAGE" "$RESET"

    printf '%b' "$C_GREEN"

    for ((i=0; i<filled; i++)); do
        printf '━'
    done

    printf '%b' "$RESET"

    for ((i=filled; i<BAR_WIDTH; i++)); do
        printf '━'
    done

    printf '  %s%%\n' "$FS_PERCENT"
}

print_inodes() {
    local line
    local total
    local used
    local free
    local percent

    line=$(df -Pi "$MOUNT" 2>/dev/null | tail -1)

    total=$(awk '{print $2}' <<< "$line")
    used=$(awk '{print $3}' <<< "$line")
    free=$(awk '{print $4}' <<< "$line")
    percent=$(awk '{print $5}' <<< "$line")

    [[ -n "$total" ]] || total="N/A"
    [[ -n "$used" ]] || used="N/A"
    [[ -n "$free" ]] || free="N/A"
    [[ -n "$percent" ]] || percent="N/A"

    printf '\n'
    printf '%b%s Inodes%b\n' \
        "$C_CYAN" "$SYM_INODE" "$RESET"

    printf '  Total: %s\n' "$total"
    printf '  Used:  %s\n' "$used"
    printf '  Free:  %s\n' "$free"
    printf '  Usage: %s\n' "$percent"
}

print_extra() {
    if $SHOW_MOUNT; then
        printf '\n%b%s Mount:%b %s\n' \
            "$C_CYAN" "$SYM_MOUNT" "$RESET" "$MOUNT"
    fi

    if $SHOW_FS; then
        local fs

        fs=$(df -T "$MOUNT" 2>/dev/null |
            tail -1 |
            awk '{print $2}')

        [[ -n "$fs" ]] || fs="N/A"

        printf '\n%b%s Filesystem:%b %s\n' \
            "$C_CYAN" "$SYM_FS" "$RESET" "$fs"
    fi

    if $SHOW_DEVICE; then
        printf '\n%b%s Device:%b %s\n' \
            "$C_CYAN" "$SYM_DEVICE" "$RESET" "$FS_DEVICE"
    fi

    if $SHOW_TYPE; then
        local type

        type=$(df -T "$MOUNT" 2>/dev/null |
            tail -1 |
            awk '{print $2}')

        [[ -n "$type" ]] || type="N/A"

        printf '\n%bType:%b %s\n' \
            "$C_CYAN" "$RESET" "$type"
    fi

    if $SHOW_SIZE; then
        printf '\n%b%s Size:%b %s\n' \
            "$C_CYAN" "$SYM_TOTAL" "$RESET" "$FS_TOTAL_H"
    fi

    if $SHOW_LABEL && has_cmd lsblk; then
        local label

        label=$(lsblk -no LABEL "$FS_DEVICE" 2>/dev/null |
            head -1)

        [[ -n "$label" ]] || label="N/A"

        printf '\n%bLabel:%b %s\n' \
            "$C_CYAN" "$RESET" "$label"
    fi

    if $SHOW_UUID && has_cmd lsblk; then
        local uuid

        uuid=$(lsblk -no UUID "$FS_DEVICE" 2>/dev/null |
            head -1)

        [[ -n "$uuid" ]] || uuid="N/A"

        printf '\n%bUUID:%b %s\n' \
            "$C_CYAN" "$RESET" "$uuid"
    fi

    if $SHOW_PARTITION && has_cmd lsblk; then
        printf '\n%bPartition:%b\n' \
            "$C_CYAN" "$RESET"

        lsblk \
            -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT \
            "$FS_DEVICE"
    fi
}

print_io() {
    local physical="$1"

    printf '\n%bI/O%b\n' \
        "$C_CYAN" "$RESET"

    if ! has_cmd iostat; then
        printf '  %bNot available.%b\n' \
            "$C_YELLOW" "$RESET"
        printf '  Install: sudo pacman -S sysstat\n'
        return
    fi

    [[ -b "$physical" ]] || {
        printf '  %bNo physical block device.%b\n' \
            "$C_YELLOW" "$RESET"
        return
    }

    local devname
    local output
    local rps
    local wps
    local iops

    devname=$(basename "$physical")

    output=$(
        iostat -dxk "$devname" 1 2 2>/dev/null |
        awk -v d="$devname" '
            $1 == d {
                line=$0
            }
            END {
                print line
            }
        '
    )

    if [[ -z "$output" ]]; then
        printf '  %bNo I/O statistics.%b\n' \
            "$C_YELLOW" "$RESET"
        return
    fi

    rps=$(awk '{print $2}' <<< "$output")
    wps=$(awk '{print $3}' <<< "$output")

    [[ "$rps" =~ ^[0-9] ]] || rps="0.00"
    [[ "$wps" =~ ^[0-9] ]] || wps="0.00"

    iops=$(
        awk -v r="$rps" -v w="$wps" \
            'BEGIN {
                printf "%.2f", r + w
            }'
    )

    if $SHOW_READ; then
        printf '  %b%s Read:%b  %s kB/s\n' \
            "$C_GREEN" "$SYM_READ" "$RESET" "$rps"
    fi

    if $SHOW_WRITE; then
        printf '  %b%s Write:%b %s kB/s\n' \
            "$C_YELLOW" "$SYM_WRITE" "$RESET" "$wps"
    fi

    if $SHOW_IOPS; then
        printf '  IOPS: %s\n' "$iops"
    fi

    if $SHOW_LATENCY; then
        printf '  Latency: N/A ms\n'
    fi
}

print_large() {
    printf '\n%bLargest directories:%b\n' \
        "$C_CYAN" "$RESET"

    du -xhd1 "$MOUNT" 2>/dev/null |
        sort -hr |
        head -n 8
}

show_mount() {
    local target="$1"

    MOUNT="$target"

    read_fs "$target" || {
        printf '%bstorage:%b cannot read %s\n' \
            "$C_RED" "$RESET" "$target" >&2
        return 1
    }

    if $QUIET; then
        printf '%s %s %s %s%%\n' \
            "$FS_TOTAL_H" \
            "$FS_FREE_H" \
            "$FS_USED_H" \
            "$FS_PERCENT"
        return
    fi

    printf '\n'
    printf '%b%s Selected disk:%b %s\n' \
        "$C_CYAN" "$SYM_DISK" "$RESET" "$MOUNT"

    printf '\n'
    print_table

    if $SHOW_BAR; then
        print_bar
    fi

    if $SHOW_MOUNT ||
       $SHOW_FS ||
       $SHOW_DEVICE ||
       $SHOW_INODE ||
       $SHOW_LABEL ||
       $SHOW_UUID ||
       $SHOW_TYPE ||
       $SHOW_SIZE ||
       $SHOW_PARTITION
    then
        print_extra

        if $SHOW_INODE; then
            print_inodes
        fi
    fi

    if $SHOW_READ ||
       $SHOW_WRITE ||
       $SHOW_IOPS ||
       $SHOW_LATENCY
    then
        local physical

        physical=$(parent_device "$FS_DEVICE")

        if [[ -n "$physical" ]]; then
            print_io "$physical"
        fi
    fi

    if $SHOW_LARGE; then
        print_large
    fi

    printf '\n'
}

show_all() {
    has_cmd lsblk || die "lsblk is required for -a"

    local disks

    disks=$(
        lsblk -dnpo NAME,TYPE |
        awk '$2 == "disk" {print $1}'
    )

    [[ -n "$disks" ]] ||
        die "no physical disks found"

    while IFS= read -r disk; do
        [[ -b "$disk" ]] || continue

        local mounts

        mounts=$(
            lsblk -nrpo NAME,MOUNTPOINT "$disk" |
            awk '$2 != "" {print $2}' |
            sort -u
        )

        printf '\n'
        printf '%b%s Disk:%b %s\n' \
            "$C_BLUE" "$SYM_DISK" "$RESET" "$disk"

        if [[ -z "$mounts" ]]; then
            printf '  %bNo mounted filesystems.%b\n' \
                "$C_YELLOW" "$RESET"
        else
            while IFS= read -r mount; do
                [[ -n "$mount" ]] || continue
                show_mount "$mount"
            done <<< "$mounts"
        fi

        if $SHOW_READ ||
           $SHOW_WRITE ||
           $SHOW_IOPS ||
           $SHOW_LATENCY
        then
            print_io "$disk"
        fi

    done <<< "$disks"
}

if $SHOW_ALL; then
    show_all
else
    show_mount "$MOUNT"
fi
