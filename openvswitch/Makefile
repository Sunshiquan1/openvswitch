# 
# Copyright (C) 2013 Julius Schulz-Zander <julius@net.t-labs.tu-berlin.de>
# Copyright (C) 2014 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
# $Id: Makefile $

include $(TOPDIR)/rules.mk

PKG_NAME:=openvswitch

PKG_RELEASE:=conntrack
PKG_VERSION:=2.15.0
PKG_RELEASE=$(PKG_SOURCE_VERSION)
PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=COPYING
PKG_USE_MIPS16:=0

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/Sunshiquan1/ovs2.15.0
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE_VERSION:=63b0899453da247bfa23517e62c40b73762fd65e
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.gz

LIB_VERSION:=1.0.0

SUPPORTED_KERNELS:=LINUX_3_8||LINUX_3_10||LINUX_3_13||LINUX_3_14||LINUX_3_18||LINUX_4_0

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/kernel.mk
$(call include_mk, python-package.mk)

PKG_FIXUP=libtool
PKG_FIXUP:=autoreconf
PKG_FIXUP:=patch-libtool
PKG_FIXUP:=gettext-version

define Package/openvswitch/Default
  SECTION:=net
  CATEGORY:=Network
  URL:=http://openvswitch.org/
  MAINTAINER:=Alexandru Ardelean <ardeleanalex@gmail.com>
endef

define Package/openvswitch/Default/description
  Open vSwitch is a production quality, multilayer, software-based, Ethernet
  virtual switch. It is designed to enable massive network automation through
  programmatic extension, while still supporting standard management interfaces
  and protocols (e.g. NetFlow, sFlow, SPAN, RSPAN, CLI, LACP, 802.1ag). In
  addition, it is designed to support distribution across multiple physical
  servers similar to VMware's vNetwork distributed vswitch or Cisco's Nexus
  1000V.
endef

define Package/openvswitch
  $(call Package/openvswitch/Default)
  TITLE:=Open vSwitch Userspace Package
  DEPENDS:=+libpcap +libopenssl +librt +libatomic +kmod-openvswitch @($(SUPPORTED_KERNELS))
endef

define Package/openvswitch/description
  Provides the main userspace components required for Open vSwitch to function.
endef

define Package/openvswitch-python
  $(call Package/openvswitch/Default)
  TITLE:=Open vSwitch Python Support
  DEPENDS:=@PACKAGE_openvswitch +PACKAGE_openvswitch:openvswitch +python
endef

define Package/openvswitch-python/description
  Provides bindings and libraries for using Python to manipulate/work with Open vSwitch.
endef

define Package/openvswitch-ipsec
  $(call Package/openvswitch/Default)
  TITLE:=Open vSwitch Userspace Package
  DEPENDS:=@PACKAGE_openvswitch +PACKAGE_openvswitch:openvswitch-python
endef

define Package/openvswitch-ipsec/description
  The ovs-monitor-ipsec script provides support for encrypting GRE tunnels with 
  IPsec.
endef

define Package/openvswitch-benchmark
  $(call Package/openvswitch/Default)
  TITLE:=Open vSwitch Userspace Package
  DEPENDS:=@PACKAGE_openvswitch +PACKAGE_openvswitch:openvswitch
endef

define Package/openvswitch-benchmark/description
  Utility for running OpenVSwitch benchmarking
endef

define KernelPackage/openvswitch
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Support
  TITLE:=Open vSwitch Kernel Package
  KCONFIG:=CONFIG_BRIDGE
  DEPENDS:=+kmod-l2tp +kmod-ipt-conntrack-extra +kmod-nf-conntrack-netlink +kmod-stp \
			+kmod-ipv6 +kmod-gre +kmod-lib-crc32c +kmod-vxlan +kmod-nf-conntrack @($(SUPPORTED_KERNELS))
  FILES:= \
	$(PKG_BUILD_DIR)/datapath/linux/openvswitch.$(LINUX_KMOD_SUFFIX)
  AUTOLOAD:=$(call AutoLoad,21,openvswitch)
endef

define KernelPackage/openvswitch/description
  This package contains the Open vSwitch kernel moodule and bridge compat
  module. Furthermore, it supports OpenFlow.
endef

CONFIGURE_ARGS += --with-linux=$(LINUX_DIR)
CONFIGURE_ARGS += --with-rundir=/var/run
CONFIGURE_ARGS += --enable-ndebug
CONFIGURE_ARGS += --disable-ssl
CONFIGURE_ARGS += --enable-shared
CONFIGURE_ARGS += --with-dbdir=/tmp/openvswitch

TARGET_CFLAGS += -flto

#PKG_FIXUP:=autoreconf

define Build/Configure
	(cd $(PKG_BUILD_DIR); \
		autoreconf -v --install --force || exit 1 \
	);
	$(call Build/Configure/Default,$(CONFIGURE_ARGS))
endef

KCFLAGS=
ifeq ($(CONFIG_GCC_VERSION_4_9),y)
KCFLAGS:=-Wno-error=date-time
endif

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) \
		$(TARGET_CONFIGURE_OPTS) \
		CFLAGS="-I$(PKG_BUILD_DIR)/lib $(TARGET_CFLAGS) -std=gnu99" \
		LDFLAGS="-L$(PKG_BUILD_DIR)/lib $(TARGET_LDFLAGS)" \
		LDFLAGS_MODULES="$(TARGET_LDFLAGS) -L$(PKG_BUILD_DIR)/lib" \
		STAGING_DIR="$(STAGING_DIR)" \
		DESTDIR="$(PKG_INSTALL_DIR)/usr" \
		CROSS_COMPILE="$(TARGET_CROSS)" \
		ARCH="$(LINUX_KARCH)" \
		SUBDIRS="$(PKG_BUILD_DIR)/datapath/linux" \
		PATH="$(TARGET_PATH)" \
		EXTRA_CFLAGS="$(KCFLAGS)" \
		KCC="$(KERNEL_CC)"
endef

#define Package/openvswitch/install
#	$(INSTALL_DIR) $(1)/etc/openvswitch
define Package/openvswitch/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/etc/init.d/openvswitch.init $(1)/etc/init.d/openvswitch

	$(INSTALL_DIR) $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/lib/.libs/libsflow.so.$(LIB_VERSION)  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/lib/.libs/libopenvswitch.so.$(LIB_VERSION)  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/ofproto/.libs/libofproto.so.$(LIB_VERSION)  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/ovsdb/.libs/libovsdb.so.$(LIB_VERSION)  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/lib/.libs/libsflow.so.1  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/lib/.libs/libopenvswitch.so.1  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/ofproto/.libs/libofproto.so.1  $(1)/usr/lib/
	$(CP) $(PKG_BUILD_DIR)/ovsdb/.libs/libovsdb.so.1  $(1)/usr/lib/

	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-appctl $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-ofctl $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/ovsdb/.libs/ovsdb-client $(1)/usr/bin/

	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-dpctl $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-vsctl $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/ovsdb/.libs/ovsdb-tool $(1)/usr/bin/

	$(INSTALL_DIR) $(1)/usr/sbin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/vswitchd/.libs/ovs-vswitchd $(1)/usr/sbin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/ovsdb/.libs/ovsdb-server $(1)/usr/sbin/

	$(INSTALL_DIR) $(1)/usr/share/openvswitch/
	$(INSTALL_CONF) $(PKG_BUILD_DIR)/vswitchd/vswitch.ovsschema $(1)/usr/share/openvswitch/
endef

define Package/openvswitch-python/install
	$(INSTALL_DIR) $(1)/usr/lib/python$(PYTHON_VERSION)/
	$(CP) $(PKG_BUILD_DIR)/python/ovs/ $(1)/usr/lib/python$(PYTHON_VERSION)/
endef

define Package/openvswitch-ipsec/install
	$(INSTALL_DIR) $(1)/usr/sbin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/debian/ovs-monitor-ipsec $(1)/usr/sbin/
endef

define Package/openvswitch-benchmark/install
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utilities/.libs/ovs-benchmark $(1)/usr/bin/
endef

define Package/openvswitch/postinst
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] || /etc/init.d/openvswitch enable || true
endef

$(eval $(call BuildPackage,openvswitch))
$(eval $(call BuildPackage,openvswitch-python))
$(eval $(call BuildPackage,openvswitch-ipsec))
$(eval $(call BuildPackage,openvswitch-benchmark))
$(eval $(call KernelPackage,openvswitch))

