#
# Copyright (C) 2013-2016 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=rpcd
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL=$(PROJECT_GIT)/project/rpcd.git
PKG_SOURCE_DATE:=2018-11-28
PKG_SOURCE_VERSION:=3aa81d0dfae167eccc26203bd0c96f3e3450f253
PKG_MAINTAINER:=Jo-Philipp Wich <jo@mein.io>
PKG_MIRROR_HASH:=1befc5e1793a687e7a37b4f4d611e5f95aef4b79ad4b288c4dcb4c74d212509b

PKG_LICENSE:=ISC
PKG_LICENSE_FILES:=

PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/cmake.mk

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_BUILD_DIR)/include/rpcd $(1)/usr/include/
endef

define Package/rpcd/default
  SECTION:=utils
  CATEGORY:=Base system
  TITLE:=OpenWrt ubus RPC backend server
  DEPENDS:=+libubus +libubox
endef

define Package/rpcd
  $(Package/rpcd/default)
  DEPENDS+= +libuci +libblobmsg-json
endef

define Package/rpcd/description
 This package provides the UBUS RPC backend server to expose various
 functionality to frontend programs via JSON-RPC.
endef

define Package/rpcd/conffiles
/etc/config/rpcd
endef

define Package/rpcd-mod-lepton/config
	if PACKAGE_rpcd-mod-lepton
		config PACKAGE_rpcd-mod-lepton-superuser
		bool "add superuser in ubus for repeater"
		default "n"
	endif
endef

TARGET_LDFLAGS += -lcrypt

define Package/rpcd/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/rpcd.init $(1)/etc/init.d/rpcd
	$(INSTALL_DIR) $(1)/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/rpcd $(1)/sbin/rpcd
	$(INSTALL_DIR) $(1)/usr/share/rpcd/acl.d
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/unauthenticated.json $(1)/usr/share/rpcd/acl.d/unauthenticated.json
ifdef CONFIG_PACKAGE_rpcd-mod-lepton-superuser
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/superuser.json $(1)/usr/share/rpcd/acl.d/superuser.json
endif
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/rpcd.config $(1)/etc/config/rpcd
endef


# 1: plugin name
# 2: extra dependencies
# 3: plugin title/description
define BuildPlugin

  PKG_CONFIG_DEPENDS += CONFIG_PACKAGE_luci-rpc-mod-$(1)

  define Package/rpcd-mod-$(1)
    $(Package/rpcd/default)
    TITLE+= ($(1) plugin)
    DEPENDS+=rpcd $(2)
  endef

  define Package/rpcd-mod-$(1)/description
    $(3)
  endef

  define Package/rpcd-mod-$(1)/install
	$(INSTALL_DIR) $$(1)/usr/lib/rpcd
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/$(1).so $$(1)/usr/lib/rpcd/
  endef

  $$(eval $$(call BuildPackage,rpcd-mod-$(1)))

endef

$(eval $(call BuildPackage,rpcd))
$(eval $(call BuildPlugin,lepton,,lepton test.))
$(eval $(call BuildPlugin,file,,Provides ubus calls for file and directory operations.))
$(eval $(call BuildPlugin,rpcsys,,Provides ubus calls for sysupgrade and password changing.))
$(eval $(call BuildPlugin,iwinfo,+libiwinfo,Provides ubus calls for accessing iwinfo data.))
