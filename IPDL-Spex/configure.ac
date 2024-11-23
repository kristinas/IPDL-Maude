AC_PREREQ([2.69])
AC_INIT([SpeX], [23.12])

AC_CHECK_PROGS([M3], [maude])
if test -z "${M3}" ; then
    AC_MSG_ERROR([Could not find Maude.])
fi

M3_version=`${M3} --version 2> /dev/null`
case ${M3_version} in
    3.[[234]]*)
        AC_MSG_NOTICE([Found maude version ${M3_version}])
        ;;
    *)
        AC_MSG_ERROR([Maude 3.2 or newer is required in order to run SpeX.])
        ;;
esac

AM_INIT_AUTOMAKE
AC_CONFIG_FILES([Makefile bin/Makefile src/Makefile])
AC_OUTPUT
