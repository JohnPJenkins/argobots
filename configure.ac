# -*- Mode: Autoconf; -*-
#
# See COPYRIGHT in top-level directory.
#

AC_PREREQ(2.63)

m4_include([maint/version.m4])
AC_INIT([argobots],ABT_VERSION_m4)

ABT_VERSION=ABT_VERSION_m4
AC_SUBST([ABT_VERSION])

libabt_so_version="libabt_so_version_m4"
AC_SUBST([libabt_so_version])

# Produce a numeric version assuming the following format:
# Version: [MAJ].[MIN].[REV][EXT][EXT_NUMBER]
# Example: 1.0.7rc1 has
#          MAJ = 1
#          MIN = 0
#          REV = 7
#          EXT = rc
#          EXT_NUMBER = 1
#
# Converting to numeric version will convert EXT to a format number:
#          ALPHA (a) = 0
#          BETA (b)  = 1
#          RC (rc)   = 2
#          PATCH (p) = 3
# Regular releases are treated as patch 0
#
# Numeric version will have 1 digit for MAJ, 2 digits for MIN,
# 2 digits for REV, 1 digit for EXT and 2 digits for EXT_NUMBER.
changequote(<<,>>)
V1=`expr $ABT_VERSION : '\([0-9]*\)\.[0-9]*\.*[0-9]*[a-zA-Z]*[0-9]*'`
V2=`expr $ABT_VERSION : '[0-9]*\.\([0-9]*\)\.*[0-9]*[a-zA-Z]*[0-9]*'`
V3=`expr $ABT_VERSION : '[0-9]*\.[0-9]*\.*\([0-9]*\)[a-zA-Z]*[0-9]*'`
V4=`expr $ABT_VERSION : '[0-9]*\.[0-9]*\.*[0-9]*\([a-zA-Z]*\)[0-9]*'`
V5=`expr $ABT_VERSION : '[0-9]*\.[0-9]*\.*[0-9]*[a-zA-Z]*\([0-9]*\)'`
changequote([,])

if test "$V2" -le 9 ; then V2="0$V2" ; fi
if test "$V3" = "" ; then V3="0"; fi
if test "$V3" -le 9 ; then V3="0$V3" ; fi
if test "$V4" = "a" ; then
    V4=0
elif test "$V4" = "b" ; then
    V4=1
elif test "$V4" = "rc" ; then
    V4=2
elif test "$V4" = "" ; then
    V4=3
    V5=0
elif test "$V4" = "p" ; then
    V4=3
fi
if test "$V5" -le 9 ; then V5="0$V5" ; fi

ABT_NUMVERSION=`expr $V1$V2$V3$V4$V5 + 0`
AC_SUBST(ABT_NUMVERSION)

AC_CONFIG_AUX_DIR(m4)
AC_CONFIG_MACRO_DIR(m4)
AC_PROG_CC
AC_HEADER_STDC

dnl ----------------------------------------------------------------------------
dnl check the environment and the function availability
dnl ----------------------------------------------------------------------------
# compute canonical system types
AC_CANONICAL_HOST

# check architecture and OS for fcontext
fctx_arch_bin=""
AS_CASE([$host_cpu],
    [i?86],   [AS_CASE([$host_os],
              [linux*],  [fctx_arch_bin="i386_sysv_elf_gas"],
              [darwin*], [fctx_arch_bin="i386_sysv_macho_gas"])],
    [x86_64], [AS_CASE([$host_os],
              [linux*],  [fctx_arch_bin="x86_64_sysv_elf_gas"],
              [darwin*], [fctx_arch_bin="x86_64_sysv_macho_gas"])],
    [p*pc32], [AS_CASE([$host_os],
              [linux*],  [fctx_arch_bin="ppc32_sysv_elf_gas"],
              [darwin*], [fctx_arch_bin="ppc32_sysv_macho_gas"])],
    [p*pc64], [AS_CASE([$host_os],
              [linux*],  [fctx_arch_bin="ppc64_sysv_elf_gas"],
              [darwin*], [fctx_arch_bin="ppc64_sysv_macho_gas"])])
AC_SUBST([fctx_arch_bin])
AM_SUBST_NOTMAKE([fctx_arch_bin])

# check pthread
AC_CHECK_HEADERS(pthread.h)
AC_CHECK_LIB(pthread, pthread_join)

# check pthread_barrier
AC_CHECK_FUNCS(pthread_barrier_init)

# check timer functions
# for clock_gettime and clock_getres, we need to search them from librt or
# libposix4 because they may not be included in the standard library.
AC_SEARCH_LIBS([clock_gettime], [rt posix4])
AC_SEARCH_LIBS([clock_getres], [rt posix4])
if test "$ac_cv_search_clock_gettime" != "no" -a \
        "$ac_cv_search_clock_getres" != "no" ; then
    AC_CHECK_FUNCS(clock_gettime clock_getres)
fi
AC_CHECK_FUNCS(mach_absolute_time gettimeofday)
if test "x$ac_cv_func_clock_gettime" = "xyes" -a \
        "x$ac_cv_func_clock_getres" = "xyes" ; then
    timer_type=clock_gettime
elif test "$ac_cv_func_mach_absolute_time" = "yes" ; then
    timer_type=mach_absolute_time
elif test "$ac_cv_func_gettimeofday" = "yes" ; then
    timer_type=gettimeofday
fi
if test -z "$timer_type" ; then
    AC_MSG_ERROR([No timer function found])
fi

# check math library
AC_CHECK_LIB([m], [log10])

# check if MAP_ANONYMOUS or MAP_ANON is defined
AC_TRY_COMPILE([
#include <stdio.h>
#include <sys/mman.h>
#define FLAGS   (MAP_PRIVATE | MAP_ANONYMOUS)
],[void *ptr = mmap(NULL, 4096, PROT_READ, FLAGS, 0, 0);],
[have_map_anonymous=yes],[have_map_anonymous=no])
if test "$have_map_anonymous" = "yes" ; then
AC_DEFINE(HAVE_MAP_ANONYMOUS, 1, [Define if MAP_ANONYMOUS is defined])
else
AC_TRY_COMPILE([
#include <stdio.h>
#include <sys/mman.h>
#define FLAGS   (MAP_PRIVATE | MAP_ANON)
],[void *ptr = mmap(NULL, 4096, PROT_READ, FLAGS, 0, 0);],
[have_map_anon=yes],[have_map_anon=no])
AS_IF([test "$have_map_anon" = "yes"],
    AC_DEFINE(HAVE_MAP_ANON, 1, [Define if MAP_ANON is defined])])
fi

if test "$have_map_anonymous" = "yes" -o "$have_map_anon" = "yes" ; then
# check if MAP_HUGETLB is supported in mmap
AC_TRY_COMPILE([
#include <stdio.h>
#include <sys/mman.h>
#ifndef MAP_ANONYMOUS
#define MAP_ANONYMOUS   MAP_ANON
#endif
#define PROTS       (PROT_READ | PROT_WRITE)
#define FLAGS_HP    (MAP_PRIVATE | MAP_ANONYMOUS | MAP_HUGETLB)
#define PAGESIZE    (2 * 1024 * 1024)
],[void *ptr = mmap(NULL, PAGESIZE, PROTS, FLAGS_HP, 0, 0);],
[AC_DEFINE(HAVE_MAP_HUGETLB, 1, [Define if MAP_HUGETLB is supported])],,)
fi
dnl ----------------------------------------------------------------------------

dnl ----------------------------------------------------------------------------
dnl setup top-level argument handling
dnl ----------------------------------------------------------------------------
# debug options
AC_ARG_ENABLE([debug],
[  --enable-debug@<:@=OPTS@:>@   control the level of debugging. "OPTS" is a list of
                          comma separated names below. Default is "yes".
                           none - no debugging
                           yes  - add compiler flags, -g -O0 -Wall
                           log  - enable debug event logging
                           all  - all of the above choices],,
[enable_debug=none])

# strip off multiple options, separated by commas
save_IFS="$IFS"
IFS=","
for option in $enable_debug ; do
    case "$option" in
        no|none)
        ;;
        yes)
            debug_flags=yes
        ;;
        log)
            debug_log=yes
        ;;
        all)
            debug_flags=yes
            debug_log=yes
        ;;
        *)
            IFS=$save_IFS
            AC_MSG_WARN([Unknown value $option for enable-debug])
            IFS=","
        ;;
    esac
done
IFS="$save_IFS"

AS_IF([test "x$debug_flags" = "xyes"], [CFLAGS="$CFLAGS -g -O0 -Wall"])
AS_IF([test "x$debug_log" = "xyes"],
    [AC_DEFINE(ABT_CONFIG_USE_DEBUG_LOG, 1, [Define to enable debug logging])])

AC_ARG_ENABLE([affinity],
    AS_HELP_STRING([--enable-affinity],
        [enable ES affinity]),,
        [enable_affinity=no])
AS_IF([test "x$enable_affinity" != "xno"], [
       AC_CHECK_LIB(pthread, pthread_setaffinity_np,
                    [AC_DEFINE(HAVE_PTHREAD_SETAFFINITY_NP, 1,
                     [Define if pthread_setaffinity_np is available])])
       ])

# set CFLAGS for the performance optimization
AC_ARG_ENABLE([perf-opt],
    AS_HELP_STRING([--enable-perf-opt], [enable performance optimization]))
if test "x$enable_perf_opt" = "xyes"; then
    if test "x$GCC" = "xyes"; then
        CFLAGS="-O3 -ftls-model=initial-exec"
    fi
fi

# enable valgrind support if requested
AC_ARG_ENABLE([valgrind],
    AS_HELP_STRING([--enable-valgrind], [enable valgrind support]))
AS_IF([test "x$enable_valgrind" = "xyes"], [
       AC_DEFINE(HAVE_VALGRIND_SUPPORT, 1, [Define valgrind support])
       ])

# Disable checks if requested
AC_ARG_ENABLE([checks],
    AS_HELP_STRING([--disable-checks],
        [disable error and pool access checks (unsafe mode)]))

AC_ARG_ENABLE([error-check],
    AS_HELP_STRING([--disable-error-check], [disable error checks]),,
    [enable_error_check=yes])
AS_IF([test "x$enable_error_check" != "xyes" -o "x$enable_checks" = "xno"],
    [AC_DEFINE(ABT_CONFIG_DISABLE_ERROR_CHECK, 1,
        [Define to disable error check])
     ABT_NULL="1"],
    [ABT_NULL="0"])
AC_SUBST([ABT_NULL])

AC_ARG_ENABLE([pool-producer-check],
    AS_HELP_STRING([--disable-pool-producer-check], [disable pool producer check]),,
    [enable_pool_producer_check=yes])
AS_IF([test "x$enable_pool_producer_check" != "xyes" -o "x$enable_checks" = "xno"],
    [AC_DEFINE(ABT_CONFIG_DISABLE_POOL_PRODUCER_CHECK, 1,
        [Define to disable pool producer check])])

AC_ARG_ENABLE([pool-consumer-check],
    AS_HELP_STRING([--disable-pool-consumer-check], [disable pool consumer check]),,
    [enable_pool_consumer_check=yes])
AS_IF([test "x$enable_pool_consumer_check" != "xyes" -o "x$enable_checks" = "xno"],
    [AC_DEFINE(ABT_CONFIG_DISABLE_POOL_CONSUMER_CHECK, 1,
        [Define to disable pool consumer check])])

AM_CONDITIONAL([ABT_CONFIG_DISABLE_POOL_ACCESS_CHECK],
    [test "x$enable_checks" = "xno" -o "x$enable_pool_producer_check" != "xyes" -o "x$enable_pool_consumer_check" != "xyes"])

AC_ARG_ENABLE([fcontext],
    AS_HELP_STRING([--disable-fcontext],
        [do not use fcontext even though it is supported. If you disable
         fcontext, ucontext in libc is used.]),,
        [enable_fcontext=yes])
# if fcontext supports this platform and the user does not specify
# --disable-fcontext, we use fcontext instead of ucontext.
AM_CONDITIONAL([ABT_USE_FCONTEXT],
               [test "x$fctx_arch_bin" != "x" -a "x$enable_fcontext" != "xno"])
AS_IF([test "x$fctx_arch_bin" != "x" -a "x$enable_fcontext" != "xno"],
      [AC_DEFINE(ABT_CONFIG_USE_FCONTEXT, 1, [Define to use fcontext])])

AC_ARG_ENABLE([preserve-fpu],
    AS_HELP_STRING([--disable-preserve-fpu],
        [do not preserve fpu registers when fcontext is used]),,
        [enable_preserve_fpu=yes])
AS_IF([test "x$enable_preserve_fpu" != "xno"],
      [AC_DEFINE(ABTD_FCONTEXT_PRESERVE_FPU, 1,
                 [Define to 1 if we preserve fpu registers])],
      [AC_DEFINE(ABTD_FCONTEXT_PRESERVE_FPU, 0)])

# Memory pool is enabled by default.
AC_ARG_ENABLE([mem-pool],
    AS_HELP_STRING([--disable-mem-pool],
        [do not use memory pools for ULT and tasklet creation]),,
        [enable_mem_pool=yes])
AS_IF([test "x$enable_mem_pool" != "xno"],
      [AC_DEFINE(ABT_CONFIG_USE_MEM_POOL, 1,
          [Define to use memory pools for ULT and tasklet creation])])

# Aligned memory allocation is enabled by default.
AC_ARG_ENABLE([aligned-alloc],
    AS_HELP_STRING([--disable-aligned-alloc],
        [do not use aligned memory allocation]),,
        [enable_aligned_alloc=yes])
AS_IF([test "x$enable_aligned_alloc" != "xno"],
    [AC_DEFINE(ABT_CONFIG_USE_ALIGNED_ALLOC, 1,
        [Define to allocate objects aligned to the cache line size])])

# libhugetlbfs
AC_ARG_WITH([hugetlbfs],
    AS_HELP_STRING([--with-hugetlbfs=PATH],
        [specify path where hugetlbfs include directory and lib directory can be found]))
if test "x$with_hugetlbfs" != "x"; then
    CFLAGS="-I$with_hugetlbfs/include $CFLAGS"
    LDFLAGS="-L$with_hugetlbfs/lib64 -lhugetlbfs $LDFLAGS"
fi
AC_CHECK_LIB(hugetlbfs, get_huge_pages)

# Power management events from outside
AC_ARG_ENABLE([power-event],
    AS_HELP_STRING([--enable-power-event],
        [enable handling power management events]))
AS_IF([test "x$enable_power_event" = "xyes"],
      [AC_DEFINE(ABT_CONFIG_HANDLE_POWER_EVENT, 1,
                 [Define to handle power management events])])

# Publish execution information
AC_ARG_ENABLE([publish-info],
    AS_HELP_STRING([--enable-publish-info],
        [enable publishing execution information]))
AS_IF([test "x$enable_publish_info" = "xyes"],
      [AC_DEFINE(ABT_CONFIG_PUBLISH_INFO, 1,
                 [Define to publish execution information])])

# BEACON path
AC_ARG_WITH([beacon],
    AS_HELP_STRING([--with-beacon=PATH],
        [specify path where beacon include directory and lib directory can be found]))
if test "x$with_beacon" != "x"; then
    CFLAGS="-I$with_beacon/include $CFLAGS"
    LDFLAGS="-L$with_beacon/lib -lbeacon $LDFLAGS"
fi
AC_CHECK_HEADERS(beacon.h)
default_pub_filename="stdout"
AC_CHECK_LIB(beacon, BEACON_Publish, [default_pub_filename="beacon"])
AC_DEFINE_UNQUOTED(ABT_CONFIG_DEFAULT_PUB_FILENAME, ["$default_pub_filename"],
                   [Default filename for publishing the performance data])

# INTERCOOLR path
AC_ARG_WITH([intercoolr],
    AS_HELP_STRING([--with-intercoolr=PATH],
        [specify path where intercoolr include directory and lib directory can be found]))
if test "x$with_intercoolr" != "x"; then
    CFLAGS="-I$with_intercoolr/include $CFLAGS"
    LDFLAGS="-L$with_intercoolr/lib -lintercoolr $LDFLAGS"
fi
AC_CHECK_HEADERS(raplreader.h)
AC_CHECK_LIB(intercoolr, raplreader_init)

# Disable features
AC_ARG_ENABLE([thread-cancel],
    AS_HELP_STRING([--disable-thread-cancel], [disable ULT cancellation]),,
    [enable_thread_cancel=yes])
AS_IF([test "x$enable_thread_cancel" != "xyes"],
    [AC_DEFINE(ABT_CONFIG_DISABLE_THREAD_CANCEL, 1,
        [Define to disable ULT cancellation])])

AC_ARG_ENABLE([task-cancel],
    AS_HELP_STRING([--disable-task-cancel], [disable tasklet cancellation]),,
    [enable_task_cancel=yes])
AS_IF([test "x$enable_task_cancel" != "xyes"],
    [AC_DEFINE(ABT_CONFIG_DISABLE_TASK_CANCEL, 1,
        [Define to disable tasklet cancellation])])

AC_ARG_ENABLE([migration],
    AS_HELP_STRING([--disable-migration], [disable ULT migration]),,
    [enable_migration=yes])
AS_IF([test "x$enable_migration" != "xyes"],
    [AC_DEFINE(ABT_CONFIG_DISABLE_MIGRATION, 1,
        [Define to disable ULT migration])])

AC_ARG_ENABLE([stackable-sched],
    AS_HELP_STRING([--disable-stackable-sched], [disable stackable scheduler]),,
    [enable_stackable_sched=yes])
AS_IF([test "x$enable_stackable_sched" != "xyes"],
    [AC_DEFINE(ABT_CONFIG_DISABLE_STACKABLE_SCHED, 1,
        [Define to disable stackable scheduler])])
dnl ----------------------------------------------------------------------------

AM_INIT_AUTOMAKE([-Wall -Wno-portability-recursive -Werror foreign 1.12.3 subdir-objects])
AM_SILENT_RULES([yes])
AM_PROG_AS
AM_PROG_AR

LT_INIT

AC_PROG_INSTALL

dnl ----------------------------------------------------------------------------
dnl config headers
dnl ----------------------------------------------------------------------------
AC_CONFIG_HEADERS([src/include/abt_config.h])
AH_TOP([
/* -*- Mode: C; c-basic-offset:4 ; indent-tabs-mode:nil ; -*- */
/*
 * See COPYRIGHT in top-level directory.
 */

#ifndef ABT_CONFIG_H_INCLUDED
#define ABT_CONFIG_H_INCLUDED
])
AH_BOTTOM([
#endif /* ABT_CONFIG_H_INCLUDED */
])
dnl ----------------------------------------------------------------------------

AC_CONFIG_FILES([Makefile
                 Doxyfile
                 maint/argobots.pc
                 src/include/abt.h
                 src/Makefile
                 test/Makefile
                 test/basic/Makefile
                 test/util/Makefile
                 examples/Makefile
                 examples/dynamic-es/Makefile])
AC_OUTPUT
