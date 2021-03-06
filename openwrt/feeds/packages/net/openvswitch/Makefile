# 
# Copyright (C) 2013 Julius Schulz-Zander <julius@net.t-labs.tu-berlin.de>
# Copyright (C) 2014 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
# $Id: Makefile $

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=openvswitch

PKG_RELEASE:=1
PKG_VERSION:=2.7.2
PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=COPYING
PKG_USE_MIPS16:=0

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=http://openvswitch.org/releases/
PKG_HASH:=fe36c86ed52f6f7c17b01cdbb7ae37bf521cc5c2e50997b618f3f742485f655b

PKG_BUILD_DEPENDS:=python/host python-six/host
PKG_CONFIG_DEPENDS:=CONFIG_PACKAGE_openvswitch-python

PKG_BUILD_PARALLEL:=1
PKG_FIXUP:=autoreconf
PKG_INSTALL:=1

SUPPORTED_KERNELS:=LINUX_3_18||LINUX_4_1||LINUX_4_3||LINUX_4_4||LINUX_4_9

include $(INCLUDE_DIR)/package.mk
$(call include_mk, python-package.mk)

define Package/openvswitch/Default
  SECTION:=net
  SUBMENU:=Open vSwitch
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

define Package/openvswitch-base
  $(call Package/openvswitch/Default)
  TITLE:=Open vSwitch Userspace Package (base)
  DEPENDS:=+libpcap +libopenssl +librt +kmod-openvswitch @($(SUPPORTED_KERNELS))
endef

define Package/openvswitch-base/description
  Provides the main userspace components required for Open vSwitch to function.
  The main OVS tools (ovs-vsctl, ovs-ofctl, etc) are packaged separately
  to conserve some room and allow more configurability.
endef

define Package/openvswitch-python
  $(call Package/openvswitch/Default)
  TITLE:=Open vSwitch Python Support
  DEPENDS:=+openvswitch +PACKAGE_openvswitch-python:python +PACKAGE_openvswitch-python:python-six
endef

define Package/openvswitch-python/description
  Provides bindings and libraries for using Python to manipulate/work with Open vSwitch.
endef

OVS_BIN_TOOLS:=ovs-appctl ovs-ofctl ovs-dpctl ovs-vsctl ovsdb-client
define Package/openvswitch
  $(call Package/openvswitch/Default)
  TITLE:=Open vSwitch Userspace Package
  DEPENDS:=+openvswitch-base $(foreach t,$(OVS_BIN_TOOLS),+openvswitch-$(t))
endef

define Package/openvswitch/description
  Provides the main userspace components required for Open vSwitch to function.
  Includes also the main OVS utilities (ovs-appctl, ovs-vsctl, etc).
endef

define KernelPackage/openvswitch
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Support
  TITLE:=Open vSwitch Kernel Package
  KCONFIG:= \
	CONFIG_BRIDGE \
	CONFIG_OPENVSWITCH \
	CONFIG_OPENVSWITCH_GRE=n \
	CONFIG_OPENVSWITCH_VXLAN=n \
	CONFIG_OPENVSWITCH_GENEVE=n
  DEPENDS:= \
	@IPV6 +kmod-gre +kmod-lib-crc32c +kmod-mpls \
	+kmod-vxlan +kmod-nf-nat +kmod-nf-nat6  \
	@($(SUPPORTED_KERNELS))
  FILES:= $(LINUX_DIR)/net/openvswitch/openvswitch.ko
  AUTOLOAD:=$(call AutoLoad,21,openvswitch)
endef

define KernelPackage/openvswitch/description
  This package contains the Open vSwitch kernel moodule and bridge compat
  module. Furthermore, it supports OpenFlow.
endef

CONFIGURE_ARGS += --with-linux=$(LINUX_DIR) --with-rundir=/var/run
CONFIGURE_ARGS += --enable-ndebug
CONFIGURE_ARGS += --disable-ssl
CONFIGURE_ARGS += --enable-shared

TARGET_CFLAGS += -flto -std=gnu99

CONFIGURE_VARS += KARCH=$(LINUX_KARCH)
MAKE_FLAGS += \
	ARCH="$(LINUX_KARCH)" \
	PYTHONPATH="$(HOST_PYTHONPATH)" \
	PYTHON="$(HOST_PYTHON_BIN)"

define OvsBinUtility
  define Package/openvswitch-$(1)
     $(call Package/openvswitch/Default)
     TITLE:=$(2)
     DEPENDS:=+openvswitch-base
  endef

  define Package/openvswitch-$(1)/description
    $(2)
  endef

  define Package/openvswitch-$(1)/install
    $(INSTALL_DIR) $$(1)/usr/bin/ ;\
    $(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/$(1) $$(1)/usr/bin/
  endef
endef

define Package/openvswitch-base/install
	$(INSTALL_DIR) $(1)/etc/openvswitch

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/etc/init.d/openvswitch.init $(1)/etc/init.d/openvswitch

	$(INSTALL_DIR) $(1)/usr/lib/
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libofproto*.so*  $(1)/usr/lib/
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libopenvswitch*.so*  $(1)/usr/lib/
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libovsdb*.so*  $(1)/usr/lib/
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libsflow*.so*  $(1)/usr/lib/

	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/ovsdb-tool $(1)/usr/bin/

	$(INSTALL_DIR) $(1)/usr/sbin/
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/sbin/ovs-vswitchd $(1)/usr/sbin/
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/sbin/ovsdb-server $(1)/usr/sbin/

	$(INSTALL_DIR) $(1)/usr/share/openvswitch/
	$(INSTALL_CONF) $(PKG_INSTALL_DIR)/usr/share/openvswitch/vswitch.ovsschema $(1)/usr/share/openvswitch/
endef

define Package/openvswitch-python/install
	$(INSTALL_DIR) $(1)/usr/lib/python$(PYTHON_VERSION)/ovs
	$(CP) $(PKG_INSTALL_DIR)/usr/share/openvswitch/python/ovs/* $(1)/usr/lib/python$(PYTHON_VERSION)/ovs
endef

define Package/openvswitch/install
	:
endef

$(eval $(call OvsBinUtility,ovs-appctl,Open vSwitch app control utility))
$(eval $(call OvsBinUtility,ovs-ofctl,Open vSwitch OpenFlow control utility))
$(eval $(call OvsBinUtility,ovs-dpctl,Open vSwitch datapath management utility))
$(eval $(call OvsBinUtility,ovs-vsctl,Open vSwitch ovs-vswitchd management utility))
$(eval $(call OvsBinUtility,ovsdb-client,Open vSwitch database JSON-RPC client))

$(foreach t,$(OVS_BIN_TOOLS),$(eval $(call BuildPackage,openvswitch-$(t))))

$(eval $(call BuildPackage,openvswitch-base))
$(eval $(call BuildPackage,openvswitch-python))
$(eval $(call BuildPackage,openvswitch))
$(eval $(call KernelPackage,openvswitch))

