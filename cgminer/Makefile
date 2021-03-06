#
# Author: Xiangfu Liu <xiangfu@openmobilefree.net>
# Address: 12h6gdGnThW385JaX1LRMA8cXKmbYRTP8Q
#
# This is free and unencumbered software released into the public domain.
# For details see the UNLICENSE file at the root of the source tree.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=cgminer

ifeq ($(CONFIG_CGMINER_AVALON2),y)
PKG_VERSION:=4.0.0
PKG_REV:=2f454d8616fbc5c371849062aaf24c2bee51d831
else
PKG_VERSION:=4.2.0
PKG_REV:=e8da9dbe5d0bb70338a927147d416d1985d325d1
endif

PKG_RELEASE:=1
PKG_INSTALL:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_REV).tar.bz2
PKG_SOURCE_URL:=git://github.com/ckolivas/cgminer.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE_VERSION:=$(PKG_REV)

PKG_FIXUP:=autoreconf

include $(INCLUDE_DIR)/package.mk

define Package/cgminer
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=cgminer
  URL:=https://github.com/ckolivas/cgminer
ifeq ($(CONFIG_TARGET_brcm2708_RaspberryPi),y)
  DEPENDS:=+libcurl +libpthread +jansson +udev +libncurses
else
  DEPENDS:=+libcurl +libpthread +jansson +udev
endif
endef

define Package/cgminer/description
Cgminer is a multi-threaded multi-pool GPU, FPGA and CPU miner with ATI GPU
monitoring, (over)clocking and fanspeed support for bitcoin and derivative
coins. Do not use on multiple block chains at the same time!
endef

define Package/cgminer/config
  menu "Configuration"
  depends on PACKAGE_cgminer
  source "$(SOURCE)/Config.in"
  endmenu
endef

CONFIGURE_ARGS += --disable-adl
TARGET_LDFLAGS += -Wl,-rpath-link=$(STAGING_DIR)/usr/lib

ifeq ($(CONFIG_CGMINER_AVALON2),)
	CONFIGURE_ARGS += --without-curses --enable-avalon
else
	CONFIGURE_ARGS += --enable-avalon2

define Build/Prepare
	$(call Build/Prepare/Default)

	wget -O $(PKG_BUILD_DIR)/0001-avalon2.patch \
		"https://raw2.github.com/BitSyncom/avalon-extras/master/cgminer-patches/0001-avalon2.patch"
	(cd $(PKG_BUILD_DIR); patch -Np1 < 0001-avalon2.patch;)
endef

endif

define Build/Compile
  $(call Build/Compile/Default)
  (cd $(PKG_BUILD_DIR) && \
    $(TARGET_CC) -Icompat/jansson -Icompat/libusb-1.0/libusb \
    api-example.c -o cgminer-api;)
endef

define Package/cgminer/install
  $(INSTALL_DIR) $(1)/usr/bin
  $(INSTALL_DIR) $(1)/etc/init.d
  $(INSTALL_DIR) $(1)/etc/config

  $(INSTALL_BIN) $(PKG_BUILD_DIR)/cgminer-api	   $(1)/usr/bin
  $(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/cgminer  $(1)/usr/bin
  $(INSTALL_BIN) $(FILES_DIR)/cgminer-monitor        $(1)/usr/bin
ifeq ($(CONFIG_CGMINER_AVALON2),)
  $(INSTALL_BIN) $(FILES_DIR)/cgminer.init           $(1)/etc/init.d/cgminer
  $(CP)          $(FILES_DIR)/cgminer.config         $(1)/etc/config/cgminer
else
  $(INSTALL_BIN) $(FILES_DIR)/cgminer.avalon2.init   $(1)/etc/init.d/cgminer
  $(CP)          $(FILES_DIR)/cgminer.avalon2.config $(1)/etc/config/cgminer
endif
endef

$(eval $(call BuildPackage,cgminer))
