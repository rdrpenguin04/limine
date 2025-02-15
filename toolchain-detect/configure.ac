AC_INIT([Limine], m4_esyscmd([../version.sh]), [https://github.com/limine-bootloader/limine/issues], [limine], [https://limine-bootloader.org/])

AC_PREREQ(2.69)

BUILDDIR="$(pwd -P)"

if test -f "$BUILDDIR/$ARCHITECTURE-toolchain.mk"; then
    exit 0
fi

AC_PROG_GREP

ENFORCE_TOOLCHAIN=no
if test "x$TOOLCHAIN" = "x"; then
    TOOLCHAIN=$ARCHITECTURE-elf
else
    ENFORCE_TOOLCHAIN=yes
fi

AC_DEFUN([CC_ERROR_MSG], [
    AC_MSG_ERROR([no suitable LIMINE_CC found, run $srcdir/make_toolchain.sh or install the appropriate GCC/Clang])
])

if ! test "x$LIMINE_CC" = "x"; then
    AC_CHECK_PROG([LIMINE_CC_0], [$LIMINE_CC], [yes])
    if ! test "x$LIMINE_CC_0" = "xyes"; then
        AC_MSG_ERROR([LIMINE_CC ($LIMINE_CC) not found])
    fi
else
    if test "$TOOLCHAIN" = 'llvm'; then
        LIMINE_CC='clang'
    else
        LIMINE_CC="$TOOLCHAIN-gcc"
    fi
    AC_CHECK_PROG([LIMINE_CC_1], [$LIMINE_CC], [yes])
    if ! test "x$LIMINE_CC_1" = "xyes"; then
        if test "x$ENFORCE_TOOLCHAIN" = "xyes"; then
            CC_ERROR_MSG
        fi
        LIMINE_CC="clang"
        AC_CHECK_PROG([LIMINE_CC_2], [$LIMINE_CC], [yes])
        if ! test "x$LIMINE_CC_2" = "xyes"; then
            LIMINE_CC="gcc"
            AC_CHECK_PROG([LIMINE_CC_3], [$LIMINE_CC], [yes])
            if ! test "x$LIMINE_CC_3" = "xyes"; then
                LIMINE_CC="cc"
                AC_CHECK_PROG([LIMINE_CC_4], [$LIMINE_CC], [yes])
                if ! test "x$LIMINE_CC_4" = "xyes"; then
                    CC_ERROR_MSG
                fi
            fi
        fi
    fi
fi
AC_SUBST(LIMINE_CC, $LIMINE_CC)

if ! $LIMINE_CC --version >/dev/null 2>&1; then
    CC_ERROR_MSG
fi

if $LIMINE_CC --version | $GREP clang >/dev/null 2>&1; then
    LIMINE_CC="$LIMINE_CC --target=$ARCHITECTURE-elf"
elif $LIMINE_CC --version | $GREP 'Free Software Foundation' >/dev/null 2>&1; then
    true
else
    CC_ERROR_MSG
fi

if ! $LIMINE_CC -dumpmachine >/dev/null 2>&1; then
    CC_ERROR_MSG
fi

CC_MACHINE="$($LIMINE_CC -dumpmachine | dd bs=${#ARCHITECTURE} count=1 2>/dev/null)"

if test "x$CC_MACHINE" = "xx86_"; then
    case "$ARCHITECTURE" in
        i?86)
            CC_MACHINE="$ARCHITECTURE"
            ;;
        *)
            ;;
    esac
fi

if ! test "x$CC_MACHINE" = "x$ARCHITECTURE"; then
    CC_ERROR_MSG
fi

AC_DEFUN([GET_BINUTILS_PROG], [
    if ! test "x$LIMINE_$1" = "x"; then
        AC_CHECK_PROG([LIMINE_$1_0], [$LIMINE_$1], [yes])
        if ! test "x$LIMINE_$1_0" = "xyes"; then
            AC_MSG_ERROR([LIMINE_$1 ($LIMINE_$1) not found])
        fi
    else
        if test "$TOOLCHAIN" = 'llvm' && test '$2' = 'ld'; then
            LIMINE_$1='ld.lld'
        else
            LIMINE_$1="$TOOLCHAIN-$2"
        fi
        AC_CHECK_PROG([LIMINE_$1_1], [$LIMINE_$1], [yes])
        if ! test "x$LIMINE_$1_1" = "xyes"; then
            if test "x$ENFORCE_TOOLCHAIN" = "xyes"; then
                AC_MSG_ERROR([LIMINE_$1 ($LIMINE_$1) not found])
            fi
            if test '$2' = 'ld'; then
                LIMINE_$1='ld.lld'
            else
                LIMINE_$1='llvm-$2'
            fi
            AC_CHECK_PROG([LIMINE_$1_2], [$LIMINE_$1], [yes])
            if ! test "x$LIMINE_$1_2" = "xyes"; then
                LIMINE_$1='$2'
                AC_CHECK_PROG([LIMINE_$1_3], [$LIMINE_$1], [yes])
                if ! test "x$LIMINE_$1_3" = "xyes"; then
                    AC_MSG_ERROR([no suitable LIMINE_$1 found, run $srcdir/make_toolchain.sh or install the appropriate LLVM/GNU binutils])
                fi
            fi
        fi
    fi
    AC_SUBST(LIMINE_$1, $LIMINE_$1)
])

GET_BINUTILS_PROG(LD, ld, linker)
GET_BINUTILS_PROG(OBJCOPY, objcopy, objcopy)
GET_BINUTILS_PROG(OBJDUMP, objdump, objdump)
GET_BINUTILS_PROG(READELF, readelf, readelf)

AC_CONFIG_FILES([$ARCHITECTURE-toolchain.mk:toolchain.mk.in])
AC_OUTPUT
