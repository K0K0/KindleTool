CC?=gcc
STRIP?=strip
DEBUG_CFLAGS=-O0 -march=native -fno-omit-frame-pointer -g3
OPT_CFLAGS=-O2 -fomit-frame-pointer -march=native
K3_CFLAGS=-O2 -ffast-math -march=armv6j -mtune=arm1136jf-s -fomit-frame-pointer -fno-stack-protector -U_FORTIFY_SOURCE
# When we want to use clang's asan (http://clang.llvm.org/docs/AddressSanitizer.html), because it rocks.
ASAN_CLFAGS="-O2 -march=native -faddress-sanitizer -fno-omit-frame-pointer -g3"

# Kindle cross toolchain prefix
CROSS_PREFIX?=arm-kindle-linux-gnueabi-

SRCS=kindle_tool.c create.c convert.c

default: all

# OS & Version handling (taken from the git makefiles)
SHELL:=/bin/bash
version-inc:
	@$(SHELL) ./version.sh
-include version-inc

ifndef DESTDIR
	PREFIX:=/.
else
	ifeq "$(OSTYPE)" "Darwin"
		# Play nice with MacPorts
		PREFIX:=/opt/local
	else
		# Play nice with Linux PMS
		PREFIX:=/usr
	endif
endif
ifeq "$(OSTYPE)" "Darwin"
	DESTDIR?=/opt/local
else
	DESTDIR?=/usr/local
endif
BINDIR:=$(DESTDIR)/$(PREFIX)/bin
MANDIR:=$(DESTDIR)/$(PREFIX)/share/man/man1

ifeq "$(OSTYPE)" "Darwin"
	CPPFLAGS?=-I/opt/local/include
else
	CPPFLAGS?=-Iincludes
endif
LIBS=-larchive -lz -lcrypto
# Needed for linking statically against OpenSSL on Linux, and as-needed takes care of it when building dynamically, so it doesn't hurt ;).
ifeq "$(OSTYPE)" "Linux"
	LIBS+=-ldl
endif
# If we want to use part of gperftools (http://gperftools.googlecode.com/svn/trunk/doc/heap_checker.html for example)
#ifeq "$(OSTYPE)" "Linux"
#	LIBS+=-ltcmalloc
#endif

ifeq "$(DEBUG)" "true"
	OUT_DIR=Debug
	CFLAGS:=$(DEBUG_CFLAGS)
else
	OUT_DIR=Release
	CFLAGS?=$(OPT_CFLAGS)
endif

ifeq "$(KINDLE)" "true"
	OUT_DIR=Kindle
	CFLAGS?=$(K3_CFLAGS)
	LDFLAGS?=-Wl,-O1 -Wl,--as-needed
	CC:=$(CROSS_PREFIX)gcc
	STRIP:=$(CROSS_PREFIX)strip
endif

# Oh, OS X...
ifeq "$(OSTYPE)" "Darwin"
	STRIP_OPTS:=
else
	STRIP_OPTS:=-s
endif

# Use a different varname in order to support appending stuff here even when setting CFLAGS as an argument to make...
KT_CFLAGS:=$(CFLAGS)
KT_CFLAGS+=-pipe
# Moar warnings!
KT_CFLAGS+=-Wall -Wformat -Wformat-security
KT_CFLAGS+=-Wextra -Wunused
KT_CFLAGS+=-Wshadow
KT_CFLAGS+=-Wmissing-prototypes
KT_CFLAGS+=-Wcast-qual
# libarchive is always built with large files support, do the same to avoid issues.
KT_CFLAGS+=-D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE -D_LARGEFILE64_SOURCE
KT_CFLAGS+=-DKT_VERSION='"$(KT_VERSION)"'
# Add a user@host build tag, unless explicitly forbidden
ifndef KT_NO_USERATHOST_TAG
	KT_CFLAGS+=-DKT_USERATHOST='"$(COMPILE_BY)@$(COMPILE_HOST)"'
endif

ifndef LDFLAGS
	ifeq "$(OSTYPE)" "Darwin"
		LDFLAGS:=-L/opt/local/lib
	else
		LDFLAGS:=-Llib
		LDFLAGS+=-Wl,-O1 -Wl,--as-needed
	endif
endif

# Let's say we want to default to building stuff that will work on OS X 10.6/10.7... If I get how this stuff works, that should do it...
#ifeq "$(OSTYPE)" "Darwin"
#	# Check that we don't mess with MacPorts, which defaults to the OS version... (I'm not sure MacPorts even sets it for us, since we're not using autotools...)
#	# Although it supposedly defaults to the OS version since OS X 10.5 anyway, so...
#	ifndef MACOSX_DEPLOYMENT_TARGET
#		export MACOSX_DEPLOYMENT_TARGET=10.6
#		KT_CFLAGS+=-mmacosx-version-min=10.6
#	endif
#endif

OBJS:=$(SRCS:%.c=$(OUT_DIR)/%.o)

$(OUT_DIR)/%.o: %.c
	$(CC) $(CPPFLAGS) $(KT_CFLAGS) -o $@ -c $<

outdir:
	mkdir -p $(OUT_DIR)

ifeq "$(OSTYPE)" "Linux"
kindletool.1:
	cp -af KindleTool.1 kindletool.1
else
kindletool.1:
endif

all: outdir kindletool kindletool.1

kindletool: version-inc $(OBJS)
	$(CC) $(CPPFLAGS) $(KT_CFLAGS) $(LDFLAGS) -o$(OUT_DIR)/$@ $(OBJS) $(LIBS)

strip: all
	$(STRIP) $(STRIP_OPTS) $(OUT_DIR)/kindletool

debug:
	$(MAKE) all DEBUG=true

kindle:
	$(MAKE) strip KINDLE=true

ifeq "$(OSTYPE)" "Linux"
clean_man:
	rm -rf kindletool.1
else
clean_man:
endif

clean: clean_man
	rm -rf Release/*.o
	rm -rf Release/kindletool
	rm -rf Debug/*.o
	rm -rf Debug/kindletool
	rm -rf Kindle/*.o
	rm -rf Kindle/kindletool
	rm -rf version-inc

ifeq "$(OSTYPE)" "Linux"
install_man:
	install -d -m 755 $(MANDIR)
	install -m 644 kindletool.1 $(MANDIR)
else
install_man:
	install -d -m 755 $(MANDIR)
	install -m 644 KindleTool.1 $(MANDIR)
endif

install: all install_man
	install -d -m 755 $(BINDIR)
	install '$(OUT_DIR)/kindletool' $(BINDIR)
	install -d -m 755 $(MANDIR)
	install -m 644 kindletool.1 $(MANDIR)


.PHONY: all install install_man clean clean_man default outdir kindletool strip debug kindle
