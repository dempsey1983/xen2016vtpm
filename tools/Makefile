XEN_ROOT = $(CURDIR)/..
include $(XEN_ROOT)/tools/Rules.mk

SUBDIRS-y :=
SUBDIRS-y += include
SUBDIRS-y += libxc
SUBDIRS-$(FLASK_ENABLE) += flask
SUBDIRS-y += xenstore
SUBDIRS-y += misc
SUBDIRS-y += examples
SUBDIRS-y += hotplug
SUBDIRS-y += xentrace
SUBDIRS-$(CONFIG_XCUTILS) += xcutils
SUBDIRS-$(CONFIG_X86) += firmware
SUBDIRS-y += console
SUBDIRS-y += xenmon
SUBDIRS-y += xenstat
SUBDIRS-$(CONFIG_Linux) += memshr 
SUBDIRS-$(CONFIG_BLKTAP1) += blktap
SUBDIRS-$(CONFIG_BLKTAP2) += blktap2
SUBDIRS-$(CONFIG_NetBSD) += xenbackendd
SUBDIRS-y += libfsimage
SUBDIRS-$(CONFIG_Linux) += libvchan

# do not recurse in to a dir we are about to delete
ifneq "$(MAKECMDGOALS)" "distclean"
SUBDIRS-$(CONFIG_QEMU_TRAD) += qemu-xen-traditional-dir
SUBDIRS-$(CONFIG_QEMU_XEN) += qemu-xen-dir
endif

SUBDIRS-y += xenpmd
SUBDIRS-y += libxl
SUBDIRS-$(CONFIG_X86) += xenpaging
SUBDIRS-$(CONFIG_X86) += debugger/gdbsx
SUBDIRS-$(CONFIG_X86) += debugger/kdd
SUBDIRS-$(CONFIG_TESTS) += tests

# These don't cross-compile
ifeq ($(XEN_COMPILE_ARCH),$(XEN_TARGET_ARCH))
SUBDIRS-y += python
SUBDIRS-y += pygrub
SUBDIRS-$(OCAML_TOOLS) += ocaml
endif

ifeq ($(CONFIG_RUMP),y)
SUBDIRS-y := include libxc xenstore
endif

# For the sake of linking, set the sys-root
ifneq ($(CROSS_COMPILE),)
CROSS_BIN_PATH ?= /usr/$(CROSS_COMPILE:-=)/bin
CROSS_SYS_ROOT ?= /usr/$(CROSS_COMPILE:-=)/sys-root
export CROSS_SYS_ROOT # exported for check/funcs.sh
export CROSS_BIN_PATH # exported for cross-install.sh
endif

.PHONY: build all
build all: subdirs-all

.PHONY: install
install: subdirs-install
	$(INSTALL_DIR) $(DESTDIR)/var/xen/dump
	$(INSTALL_DIR) $(DESTDIR)/var/log/xen
	$(INSTALL_DIR) $(DESTDIR)/var/lib/xen

.PHONY: uninstall
uninstall: D=$(DESTDIR)
uninstall:
	[ -d $(D)$(XEN_CONFIG_DIR) ] && mv -f $(D)$(XEN_CONFIG_DIR) $(D)$(XEN_CONFIG_DIR).old-`date +%s` || true
	rm -rf $(D)$(CONFIG_DIR)/init.d/xendomains $(D)$(CONFIG_DIR)/init.d/xend
	rm -rf $(D)$(CONFIG_DIR)/init.d/xencommons $(D)$(CONFIG_DIR)/init.d/xen-watchdog
	rm -f  $(D)$(CONFIG_DIR)/udev/rules.d/xen-backend.rules
	rm -f  $(D)$(CONFIG_DIR)/udev/rules.d/xend.rules
	rm -f  $(D)$(SYSCONFIG_DIR)/xendomains
	rm -f  $(D)$(SBINDIR)/xendomains
	rm -f  $(D)$(SYSCONFIG_DIR)/xencommons
	rm -rf $(D)/var/lib/xen*
	rm -rf $(D)$(BINDIR)/cpuperf-perfcntr $(D)$(BINDIR)/cpuperf-xen
	rm -rf $(D)$(BINDIR)/xc_shadow
	rm -rf $(D)$(BINDIR)/pygrub
	rm -rf $(D)$(BINDIR)/setsize $(D)$(BINDIR)/tbctl
	rm -rf $(D)$(BINDIR)/xsls
	rm -rf $(D)$(BINDIR)/xenstore* $(D)$(BINDIR)/xentrace*
	rm -rf $(D)$(BINDIR)/xen-detect $(D)$(BINDIR)/xencons
	rm -rf $(D)$(BINDIR)/xenpvnetboot $(D)$(BINDIR)/qemu-*-xen
	rm -rf $(D)$(INCLUDEDIR)/xenctrl* $(D)$(INCLUDEDIR)/xenguest.h
	rm -rf $(D)$(INCLUDEDIR)/xs_lib.h $(D)$(INCLUDEDIR)/xs.h
	rm -rf $(D)$(INCLUDEDIR)/xenstore-compat/xs_lib.h $(D)$(INCLUDEDIR)/xenstore-compat/xs.h
	rm -rf $(D)$(INCLUDEDIR)/xenstore_lib.h $(D)$(INCLUDEDIR)/xenstore.h
	rm -rf $(D)$(INCLUDEDIR)/xen
	rm -rf $(D)$(INCLUDEDIR)/_libxl* $(D)$(INCLUDEDIR)/libxl*
	rm -rf $(D)$(INCLUDEDIR)/xenstat.h $(D)$(INCLUDEDIR)/xentoollog.h
	rm -rf $(D)$(LIBDIR)/libxenctrl* $(D)$(LIBDIR)/libxenguest*
	rm -rf $(D)$(LIBDIR)/libxenstore* $(D)$(LIBDIR)/libxlutil*
	rm -rf $(D)$(LIBDIR)/python/xen $(D)$(LIBDIR)/python/grub
	rm -rf $(D)$(LIBEXEC)
	rm -rf $(D)$(SBINDIR)/setmask
	rm -rf $(D)$(SBINDIR)/xen* $(D)$(SBINDIR)/netfix $(D)$(SBINDIR)/xm
	rm -rf $(D)$(SHAREDIR)/doc/xen
	rm -rf $(D)$(SHAREDIR)/xen
	rm -rf $(D)$(SHAREDIR)/qemu-xen
	rm -rf $(D)$(MAN1DIR)/xen*
	rm -rf $(D)$(MAN8DIR)/xen*

.PHONY: clean
clean: subdirs-clean

.PHONY: distclean
distclean: subdirs-distclean
	rm -rf qemu-xen-traditional-dir qemu-xen-traditional-dir-remote
	rm -rf qemu-xen-dir qemu-xen-dir-remote
	rm -rf ../config/Tools.mk config.h config.log config.status \
		config.cache autom4te.cache

ifneq ($(XEN_COMPILE_ARCH),$(XEN_TARGET_ARCH))
IOEMU_CONFIGURE_CROSS ?= --cross-prefix=$(CROSS_COMPILE) \
			 --interp-prefix=$(CROSS_SYS_ROOT)
endif

ifeq ($(XEN_TOOLS_RPATH),y)
QEMU_UPSTREAM_RPATH := -Wl,-rpath,$(LIBEXEC_LIB):$(LIBDIR)
IOEMU_EXTRA_LDFLAGS := --extra-ldflags="-Wl,-rpath,$(LIBDIR)"
else
QEMU_UPSTREAM_RPATH := -Wl,-rpath,$(LIBEXEC_LIB)
IOEMU_EXTRA_LDFLAGS :=
endif

QEMU_ROOT := $(shell if [ -d "$(QEMU_TRADITIONAL_LOC)" ]; then echo "$(QEMU_TRADITIONAL_LOC)"; else echo .; fi)
ifneq ($(QEMU_ROOT),.)
export QEMU_ROOT
endif

qemu-xen-traditional-dir-find:
	set -ex; \
	if test -d $(QEMU_TRADITIONAL_LOC); then \
		mkdir -p qemu-xen-traditional-dir; \
	else \
		export GIT=$(GIT); \
		$(XEN_ROOT)/scripts/git-checkout.sh $(QEMU_TRADITIONAL_LOC) $(QEMU_TRADITIONAL_REVISION) qemu-xen-traditional-dir; \
	fi

qemu-xen-dir-find:
	if test -d $(QEMU_UPSTREAM_LOC) ; then \
		mkdir -p qemu-xen-dir; \
	else \
		export GIT=$(GIT); \
		$(XEN_ROOT)/scripts/git-checkout.sh $(QEMU_UPSTREAM_LOC) $(QEMU_UPSTREAM_REVISION) qemu-xen-dir ; \
	fi

.PHONY: qemu-xen-traditional-dir-force-update
qemu-xen-traditional-dir-force-update: qemu-xen-traditional-dir-find
	set -ex; \
	if [ "$(QEMU_TRADITIONAL_REVISION)" ]; then \
		cd qemu-xen-traditional-dir-remote; \
		$(GIT) fetch origin; \
		$(GIT) reset --hard $(QEMU_TRADITIONAL_REVISION); \
	fi

subdir-all-qemu-xen-traditional-dir: qemu-xen-traditional-dir-find
	set -e; \
		$(buildmakevars2shellvars); \
		cd qemu-xen-traditional-dir; \
		$(QEMU_ROOT)/xen-setup \
		$(IOEMU_EXTRA_LDFLAGS) \
		--cpu=$(IOEMU_CPU_ARCH) \
		$(IOEMU_CONFIGURE_CROSS); \
		$(MAKE) all

subdir-install-qemu-xen-traditional-dir: qemu-xen-traditional-dir-find
	set -e; \
		$(buildmakevars2shellvars); \
		cd qemu-xen-traditional-dir; \
		$(QEMU_ROOT)/xen-setup \
		--extra-cflags="$(EXTRA_CFLAGS_QEMU_TRADITIONAL)" \
		$(IOEMU_EXTRA_LDFLAGS) \
		--cpu=$(IOEMU_CPU_ARCH) \
		$(IOEMU_CONFIGURE_CROSS); \
		$(MAKE) install

subdir-clean-qemu-xen-traditional-dir:
	set -e; if test -d qemu-xen-traditional-dir/.; then \
		$(MAKE) -C qemu-xen-traditional-dir clean; \
	fi

.PHONY: qemu-xen-dir-force-update
qemu-xen-dir-force-update: qemu-xen-dir-find
	set -ex; \
	if [ "$(QEMU_UPSTREAM_REVISION)" ]; then \
		cd qemu-xen-dir-remote; \
		$(GIT) fetch origin; \
		$(GIT) reset --hard $(QEMU_UPSTREAM_REVISION); \
	fi

ifeq ($(debug),y)
QEMU_XEN_ENABLE_DEBUG := --enable-debug --enable-trace-backend=stderr
else
QEMU_XEN_ENABLE_DEBUG :=
endif

ifeq ($(QEMU_STUBDOM_VTPM),y)
QEMU_TPM_ARGS="--enable-tpm"
else
QEMU_TPM_ARGS="--disable-tpm"
endif

subdir-all-qemu-xen-dir: qemu-xen-dir-find
	if test -d $(QEMU_UPSTREAM_LOC) ; then \
		source=$(QEMU_UPSTREAM_LOC); \
	else \
		source=.; \
	fi; \
	cd qemu-xen-dir; \
	$$source/configure --enable-xen --target-list=i386-softmmu \
		$(QEMU_XEN_ENABLE_DEBUG) \
		--prefix=$(LIBEXEC) \
		--libdir=$(LIBEXEC_LIB) \
		--includedir=$(LIBEXEC_INC) \
		--source-path=$$source \
		--extra-cflags="-I$(XEN_ROOT)/tools/include \
		-I$(XEN_ROOT)/tools/libxc/include \
		-I$(XEN_ROOT)/tools/xenstore/include \
		-I$(XEN_ROOT)/tools/xenstore/compat/include \
		$(EXTRA_CFLAGS_QEMU_XEN)" \
		--extra-ldflags="-L$(XEN_ROOT)/tools/libxc \
		-L$(XEN_ROOT)/tools/xenstore \
		$(QEMU_UPSTREAM_RPATH)" \
		--bindir=$(LIBEXEC_BIN) \
		--datadir=$(SHAREDIR)/qemu-xen \
		--localstatedir=$(localstatedir) \
		--disable-kvm \
                $(QEMU_TPM_ARGS) \
		--disable-docs \
		--disable-guest-agent \
		--python=$(PYTHON) \
		$(CONFIG_QEMUU_EXTRA_ARGS) \
		--cpu=$(IOEMU_CPU_ARCH) \
		$(IOEMU_CONFIGURE_CROSS); \
	$(MAKE) all

subdir-install-qemu-xen-dir: subdir-all-qemu-xen-dir
	cd qemu-xen-dir; \
	$(MAKE) install

subdir-clean-qemu-xen-dir:
	set -e; if test -d qemu-xen-dir/.; then \
		$(MAKE) -C qemu-xen-dir clean; \
	fi

subdir-clean-debugger/gdbsx subdir-distclean-debugger/gdbsx: .phony
	$(MAKE) -C debugger/gdbsx clean

subdir-install-debugger/gdbsx: .phony
	$(MAKE) -C debugger/gdbsx install

subdir-all-debugger/gdbsx: .phony
	$(MAKE) -C debugger/gdbsx all


subdir-clean-debugger/kdd subdir-distclean-debugger/kdd: .phony
	$(MAKE) -C debugger/kdd clean

subdir-install-debugger/kdd: .phony
	$(MAKE) -C debugger/kdd install

subdir-all-debugger/kdd: .phony
	$(MAKE) -C debugger/kdd all

subdir-distclean-firmware: .phony
	$(MAKE) -C firmware distclean

subtree-force-update:
ifeq ($(CONFIG_QEMU_XEN),y)
	$(MAKE) qemu-xen-dir-force-update
endif
ifeq ($(CONFIG_QEMU_TRAD),y)
	$(MAKE) qemu-xen-traditional-dir-force-update
endif
	$(MAKE) -C firmware subtree-force-update

subtree-force-update-all:
	$(MAKE) qemu-xen-dir-force-update
	$(MAKE) qemu-xen-traditional-dir-force-update
	$(MAKE) -C firmware subtree-force-update-all
