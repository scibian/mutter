AC_PREREQ(2.62)

m4_define([mutter_major_version], [3])
m4_define([mutter_minor_version], [14])
m4_define([mutter_micro_version], [4])

m4_define([mutter_version],
          [mutter_major_version.mutter_minor_version.mutter_micro_version])

m4_define([mutter_plugin_api_version], [3])

AC_INIT([mutter], [mutter_version],
        [http://bugzilla.gnome.org/enter_bug.cgi?product=mutter])

AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_SRCDIR(src/core/display.c)
AC_CONFIG_HEADERS(config.h)

AM_INIT_AUTOMAKE([1.11 foreign no-dist-gzip dist-xz tar-ustar subdir-objects])
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])],)
AM_MAINTAINER_MODE([enable])

MUTTER_MAJOR_VERSION=mutter_major_version
MUTTER_MINOR_VERSION=mutter_minor_version
MUTTER_MICRO_VERSION=mutter_micro_version
MUTTER_PLUGIN_API_VERSION=mutter_plugin_api_version
AC_SUBST(MUTTER_MAJOR_VERSION)
AC_SUBST(MUTTER_MINOR_VERSION)
AC_SUBST(MUTTER_MICRO_VERSION)
AC_SUBST(MUTTER_PLUGIN_API_VERSION)

MUTTER_PLUGIN_DIR="$libdir/$PACKAGE/plugins"
AC_SUBST(MUTTER_PLUGIN_DIR)

# Honor aclocal flags
AC_SUBST(ACLOCAL_AMFLAGS, "\${ACLOCAL_FLAGS}")

GETTEXT_PACKAGE=mutter
AC_SUBST(GETTEXT_PACKAGE)
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE,"$GETTEXT_PACKAGE",[Name of default gettext domain])

LT_PREREQ([2.2.6])
LT_INIT([disable-static])
IT_PROG_INTLTOOL([0.41])
AC_PROG_CC
AC_PROG_CC_C_O
AC_PROG_INSTALL
AC_HEADER_STDC
PKG_PROG_PKG_CONFIG([0.21])

# Sets GLIB_GENMARSHAL and GLIB_MKENUMS
AM_PATH_GLIB_2_0()

#### Integer sizes

AC_CHECK_SIZEOF(char)
AC_CHECK_SIZEOF(short)
AC_CHECK_SIZEOF(long)
AC_CHECK_SIZEOF(int)
AC_CHECK_SIZEOF(void *)
AC_CHECK_SIZEOF(long long)
AC_CHECK_SIZEOF(__int64)

## byte order
AC_C_BIGENDIAN

CANBERRA_GTK=libcanberra-gtk3
CANBERRA_GTK_VERSION=0.26

CLUTTER_PACKAGE=clutter-1.0

MUTTER_PC_MODULES="
   gtk+-3.0 >= 3.9.11
   gio-unix-2.0 >= 2.35.1
   pango >= 1.2.0
   cairo >= 1.10.0
   gsettings-desktop-schemas >= 3.7.3
   $CLUTTER_PACKAGE >= 1.19.5
   cogl-1.0 >= 1.17.1
   upower-glib >= 0.99.0
   gnome-desktop-3.0
   xcomposite >= 0.2
   xcursor
   xdamage
   xext
   xfixes
   xi >= 1.6.0
   xkbfile
   xkeyboard-config
   xkbcommon >= 0.4.3
   xkbcommon-x11
   x11-xcb
   xcb-randr
"

GLIB_GSETTINGS

AC_ARG_ENABLE(verbose-mode,
  AC_HELP_STRING([--disable-verbose-mode],
                 [disable mutter's ability to do verbose logging, for embedded/size-sensitive custom builds]),,
  enable_verbose_mode=yes)

if test x$enable_verbose_mode = xyes; then
    AC_DEFINE(WITH_VERBOSE_MODE,1,[Build with verbose mode support])
fi

AC_ARG_ENABLE(sm,
  AC_HELP_STRING([--disable-sm],
                 [disable mutter's session management support, for embedded/size-sensitive custom non-GNOME builds]),,
  enable_sm=auto)

AC_ARG_ENABLE(startup-notification,
  AC_HELP_STRING([--disable-startup-notification],
                 [disable mutter's startup notification support, for embedded/size-sensitive custom non-GNOME builds]),,
  enable_startup_notification=auto)

AC_ARG_WITH(libcanberra,
  AC_HELP_STRING([--without-libcanberra],
                 [disable the use of libcanberra for playing sounds]),,
  with_libcanberra=auto)

AC_ARG_WITH([xwayland-path],
            [AS_HELP_STRING([--with-xwayland-path], [Absolute path for an X Wayland server])],
            [XWAYLAND_PATH="$withval"],
            [XWAYLAND_PATH="$bindir/Xwayland"])

AC_ARG_ENABLE(installed_tests,
              AS_HELP_STRING([--enable-installed-tests],
                             [Install test programs (default: no)]),,
              [enable_installed_tests=no])
AM_CONDITIONAL(BUILDOPT_INSTALL_TESTS, test x$enable_installed_tests = xyes)

## here we get the flags we'll actually use

# Unconditionally use this dir to avoid a circular dep with gnomecc
GNOME_KEYBINDINGS_KEYSDIR="${datadir}/gnome-control-center/keybindings"
AC_SUBST(GNOME_KEYBINDINGS_KEYSDIR)

STARTUP_NOTIFICATION_VERSION=0.7
AC_MSG_CHECKING([Startup notification library >= $STARTUP_NOTIFICATION_VERSION])
if $PKG_CONFIG --atleast-version $STARTUP_NOTIFICATION_VERSION libstartup-notification-1.0; then
   have_startup_notification=yes
else
   have_startup_notification=no
fi
AC_MSG_RESULT($have_startup_notification)

if test x$enable_startup_notification = xyes; then
   have_startup_notification=yes
   echo "startup-notification support forced on"
elif test x$enable_startup_notification = xauto; then
   true
else
   have_startup_notification=no
fi

if test x$have_startup_notification = xyes; then
  echo "Building with libstartup-notification"
  MUTTER_PC_MODULES="$MUTTER_PC_MODULES libstartup-notification-1.0 >= $STARTUP_NOTIFICATION_VERSION"
  AC_DEFINE(HAVE_STARTUP_NOTIFICATION, , [Building with startup notification support])
else
  echo "Building without libstartup-notification"
fi

have_libcanberra=no
AC_MSG_CHECKING([libcanberra-gtk])
if test x$with_libcanberra = xno ; then
  AC_MSG_RESULT([disabled])
else
  if $PKG_CONFIG --exists $CANBERRA_GTK '>=' $CANBERRA_GTK_VERSION; then
    have_libcanberra=yes
    AC_MSG_RESULT(yes)
    MUTTER_PC_MODULES="$MUTTER_PC_MODULES $CANBERRA_GTK"
    AC_DEFINE([HAVE_LIBCANBERRA], 1, [Building with libcanberra for playing sounds])
  else
    AC_MSG_RESULT(no)
    if test x$with_libcanberra = xyes ; then
      AC_MSG_ERROR([libcanberra forced and libcanberra-gtk was not found])
    fi
  fi
fi

INTROSPECTION_VERSION=0.9.5
GOBJECT_INTROSPECTION_CHECK([$INTROSPECTION_VERSION])

if test x$found_introspection != xno; then
  AC_DEFINE(HAVE_INTROSPECTION, 1, [Define if GObject introspection is available])
  MUTTER_PC_MODULES="$MUTTER_PC_MODULES gobject-introspection-1.0"
  # Since we don't make any guarantees about stability and we don't support
  # parallel install, there's no real reason to change directories, filenames,
  # etc. as we change the Mutter tarball version. Note that this must match
  # api_version in src/Makefile.am
  META_GIR=Meta_3_0_gir
  # META_GIR=[Meta_]mutter_major_version[_]mutter_minor_version[_gir]
  AC_SUBST(META_GIR)
fi

AC_SUBST(XWAYLAND_PATH)

PKG_CHECK_MODULES(MUTTER, $MUTTER_PC_MODULES)

PKG_CHECK_MODULES(MUTTER_NATIVE_BACKEND, [clutter-egl-1.0 libdrm libsystemd libinput gbm >= 10.3], [have_native_backend=yes], [have_native_backend=no])
if test $have_native_backend = yes; then
   AC_DEFINE([HAVE_NATIVE_BACKEND],[1],[Define if you want to enable the native (KMS) backend based on systemd])
fi
AM_CONDITIONAL([HAVE_NATIVE_BACKEND],[test $have_native_backend = yes])

PKG_CHECK_MODULES(MUTTER_WAYLAND, [clutter-wayland-1.0 clutter-wayland-compositor-1.0 wayland-server >= 1.5.90], [have_wayland=yes], [have_wayland=no])
if test $have_wayland = yes; then
   AC_PATH_PROG([WAYLAND_SCANNER],[wayland-scanner],[no])
   AS_IF([test $WAYLAND_SCANNER = "no"],
     AC_MSG_ERROR([Could not find wayland-scanner in your PATH, required for parsing wayland extension protocols]))
   AC_SUBST([WAYLAND_SCANNER])

   AC_DEFINE([HAVE_WAYLAND],[1],[Define if you want to enable Wayland support])
fi
AM_CONDITIONAL([HAVE_WAYLAND],[test $have_wayland = yes])

PKG_CHECK_EXISTS([xi >= 1.6.99.1],
                 AC_DEFINE([HAVE_XI23],[1],[Define if you have support for XInput 2.3 or greater]))

# This is used for plugins
AC_SUBST(CLUTTER_PACKAGE)
PKG_CHECK_MODULES(CLUTTER, $CLUTTER_PACKAGE)

AC_PATH_XTRA

ALL_X_LIBS="$X_LIBS $X_PRE_LIBS -lX11 $X_EXTRA_LIBS"

# Check for Xinerama extension - we only support the "XFree86" style,
# and not the older Solaris-only version; recent Solaris supports the
# XFree86 style.
mutter_save_cppflags="$CPPFLAGS"
CPPFLAGS="$CPPFLAGS $X_CFLAGS"

have_xinerama=yes
AC_CHECK_LIB(Xinerama, XineramaQueryExtension,
    [AC_CHECK_HEADER(X11/extensions/Xinerama.h,
		     [X_EXTRA_LIBS="-lXinerama $X_EXTRA_LIBS"
		     if test -z "`echo $ALL_X_LIBS | grep "\-lXext" 2> /dev/null`"; then
			 X_EXTRA_LIBS="-lXext $X_EXTRA_LIBS"
		     fi],
		     have_xinerama=no,
		     [#include <X11/Xlib.h>])],
    have_xinerama=no, -lXext $ALL_X_LIBS)
AC_MSG_CHECKING(for Xinerama support)
AC_MSG_RESULT($have_xinerama)

CPPFLAGS="$mutter_save_cppflags"

if test x$have_xinerama = xno; then
   AC_MSG_ERROR([Xinerama extension was not found])
fi

AC_DEFINE_UNQUOTED([XKB_BASE], ["`$PKG_CONFIG --variable xkb_base xkeyboard-config`"],
                               [XKB base dir])

RANDR_LIBS=
found_randr=no
AC_CHECK_LIB(Xrandr, XRRUpdateConfiguration,
               [AC_CHECK_HEADER(X11/extensions/Xrandr.h,
                                RANDR_LIBS=-lXrandr found_randr=yes,,
				[#include <X11/Xlib.h>])],
               , -lXext $ALL_X_LIBS)

if test "x$found_randr" = "xyes"; then
   AC_DEFINE(HAVE_RANDR, , [Have the Xrandr extension library])
fi

MUTTER_LIBS="$MUTTER_LIBS $RANDR_LIBS $X_LIBS $X_PRE_LIBS -lX11 $X_EXTRA_LIBS -lm"

found_sm=no
case "$MUTTER_LIBS" in
 *-lSM*)
    found_sm=yes
    ;;
 *)
  AC_CHECK_LIB(SM, SmcSaveYourselfDone,
               [AC_CHECK_HEADERS(X11/SM/SMlib.h,
                                MUTTER_LIBS="-lSM -lICE $MUTTER_LIBS" found_sm=yes)],
               , $MUTTER_LIBS)
    ;;
esac

if test x$enable_sm = xno; then
   found_sm=no
fi

if test x$enable_sm = xyes; then
   if test "$found_sm" = "no"; then
      AC_MSG_ERROR([--enable-sm forced and -lSM not found])
      exit 1
   fi
fi

if test "$found_sm" = "yes"; then
   AC_DEFINE(HAVE_SM, , [Building with SM support])
fi

AM_CONDITIONAL(HAVE_SM, test "$found_sm" = "yes")

AC_PATH_PROG(ZENITY, zenity, no)
if test x"$ZENITY" = xno; then
  AC_MSG_ERROR([zenity not found in your path - needed for dialogs])
fi

AC_ARG_ENABLE(debug,
	[  --enable-debug		enable debugging],,
	enable_debug=no)
if test "x$enable_debug" = "xyes"; then
	CFLAGS="$CFLAGS -g -O"
fi

GTK_DOC_CHECK([1.15], [--flavour no-tmpl])

#### Warnings (last since -Werror can disturb other tests)

# Stay command-line compatible with the gnome-common configure option. Here
# minimum/yes/maximum are the same, however.
AC_ARG_ENABLE(compile_warnings,
  AS_HELP_STRING([--enable-compile-warnings=@<:@no/minimum/yes/maximum/error@:>@],[Turn on compiler warnings]),,
  enable_compile_warnings=error)

changequote(,)dnl
if test "$enable_compile_warnings" != no ; then
  if test "x$GCC" = "xyes"; then
    case " $CFLAGS " in
    *[\ \	]-Wall[\ \	]*) ;;
    *) CFLAGS="$CFLAGS -Wall" ;;
    esac

#   case " $CFLAGS " in
#   *[\ \	]-Wshadow[\ \	]*) ;;
#   *) CFLAGS="$CFLAGS -Wshadow" ;;
#   esac

    case " $CFLAGS " in
    *[\ \	]-Wchar-subscripts[\ \	]*) ;;
    *) CFLAGS="$CFLAGS -Wchar-subscripts" ;;
    esac

    case " $CFLAGS " in
    *[\ \	]-Wmissing-declarations[\ \	]*) ;;
    *) CFLAGS="$CFLAGS -Wmissing-declarations" ;;
    esac

    case " $CFLAGS " in
    *[\ \	]-Wmissing-prototypes[\ \	]*) ;;
    *) CFLAGS="$CFLAGS -Wmissing-prototypes" ;;
    esac

    case " $CFLAGS " in
    *[\ \	]-Wnested-externs[\ \	]*) ;;
    *) CFLAGS="$CFLAGS -Wnested-externs" ;;
    esac

    case " $CFLAGS " in
    *[\ \	]-Wpointer-arith[\ \	]*) ;;
    *) CFLAGS="$CFLAGS -Wpointer-arith" ;;
    esac

    case " $CFLAGS " in
    *[\ \	]-Wcast-align[\ \	]*) ;;
    *) CFLAGS="$CFLAGS -Wcast-align" ;;
    esac

    case " $CFLAGS " in
    *[\ \	]-Wsign-compare[\ \	]*) ;;
    *) CFLAGS="$CFLAGS -Wsign-compare" ;;
    esac

    if test "$enable_compile_warnings" = error; then
      case " $CFLAGS " in
      *[\ \	]-Werror[\ \	]*) ;;
      *) CFLAGS="$CFLAGS -Werror -Wno-error=deprecated-declarations" ;;
      esac
    fi
  fi
fi
changequote([,])dnl

AC_CONFIG_FILES([
Makefile
data/Makefile
doc/Makefile
doc/man/Makefile
doc/reference/Makefile
doc/reference/meta-docs.sgml
src/Makefile
src/libmutter.pc
src/compositor/plugins/Makefile
src/meta/meta-version.h
po/Makefile.in
])

AC_OUTPUT

if test x$enable_verbose_mode = xno; then
        echo "*** WARNING WARNING WARNING WARNING WARNING"
        echo "*** Building without verbose mode"
        echo "*** This means there's no way to debug mutter problems."
        echo "*** Please build normal desktop versions of mutter"
        echo "*** with verbose mode enabled so users can use it when they report bugs."
fi

dnl ==========================================================================
echo "
mutter-$VERSION

	prefix:                   ${prefix}
	source code location:	  ${srcdir}
	compiler:		  ${CC}

	Startup notification:     ${have_startup_notification}
	libcanberra:              ${have_libcanberra}
	Introspection:            ${found_introspection}
	Session management:       ${found_sm}
"


MUTTER_MINOR_VERSION=mutter_minor_version
if expr $MUTTER_MINOR_VERSION % 2 > /dev/null ; then
   stable_version=`expr $MUTTER_MINOR_VERSION - 1`
   echo "This is the UNSTABLE branch of mutter"
   echo -n "Use 3.$stable_version.x for stable "
   echo    "(gnome-3-$stable_version branch in git)"
else
   echo "This is the stable branch of mutter"
fi
