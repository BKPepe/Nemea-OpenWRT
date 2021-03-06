#
# Copyright (C) 2006-2013 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

ifndef include_mk
include $(TOPDIR)/feeds/packages/lang/python/python3-package.mk
include $(TOPDIR)/feeds/packages/lang/python/python3-host.mk
else
$(call include_mk, python3-package.mk)
$(call include_mk, python3-host.mk)
endif

PKG_NAME:=nemea-framework
PKG_REV:=055a202
PKG_VERSION:=2.1.1
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_VERSION:=$(PKG_REV)
PKG_SOURCE_SUBDIR:=nemea-framework-$(PKG_VERSION)
PKG_SOURCE_URL:=https://github.com/CESNET/nemea-framework
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_REV).tar.gz
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

PKG_FIXUP:=autoreconf

PKG_LICENSE:=GPLv2
PKG_LICENSE_FILES:=COPYING

PKG_MAINTAINER:=Tomas Cejka <cejkat@cesnet.cz>

PKG_INSTALL:=1
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

define Package/nemea-framework
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=NEMEA
  DEPENDS:=+libpthread +librt +libstdcpp +libopenssl
  TITLE:=NEMEA system libraries
  URL:=https://github.com/CESNET/nemea-framework
  MENU:=1
endef

define Package/nemea-framework/description
This package contains a system-independent library for the Nemea system - system for network traffic analysis and anomaly detection.
endef

define Package/nemea-framework/config
	source "$(SOURCE)/Config.in"
endef

TARGET_CFLAGS += \
	-ffunction-sections \
	-fdata-sections \
	--std=gnu99

CONFIGURE_VARS += \
	ac_cv_linux_vers=$(LINUX_VERSION) \
	ac_cv_header_libusb_1_0_libusb_h=no \
	ac_cv_netfilter_can_compile=no


CONFIGURE_ARGS += \
	--enable-shared \
	--enable-static \
	--disable-python --disable-pycommon --disable-silent-rules --disable-example --disable-tests LIBS=-lm --bindir=\$$(bindir)/nemea \
	--with-build-cc="$(HOSTCC)" \
	--with-ifcbuffersize=$(CONFIG_NEMEA_LIBTRAP_BUFFER_SIZE) \
	--with-ifcmaxclients=$(CONFIG_NEMEA_LIBTRAP_IFC_MAX_CLIENTS) \
	--without-libxml2

MAKE_FLAGS += \
	CCOPT="$(TARGET_CFLAGS) -I$(BUILD_DIR)/linux/include"

define Build/Configure
	$(call Build/Configure/Default)
endef
define Build/InstallDev
	$(INSTALL_DIR) $(2)/bin
	$(CP) $(PKG_BUILD_DIR)/unirec/ur_processor.sh $(2)/bin/

	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_INSTALL_DIR)/usr/include/libtrap* $(1)/usr/include/
	$(INSTALL_DIR) $(1)/usr/include/nemea-common
	$(CP) $(PKG_INSTALL_DIR)/usr/include/nemea-common* $(1)/usr/include/
	$(INSTALL_DIR) $(1)/usr/include/unirec/
	$(CP) $(PKG_INSTALL_DIR)/usr/include/unirec* $(1)/usr/include/

	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/lib*.{a,so*} $(1)/usr/lib/

	$(INSTALL_DIR) $(1)/usr/lib/pkgconfig
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/pkgconfig/*.pc $(1)/usr/lib/pkgconfig
endef

define Package/nemea-framework/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/lib*.so* $(1)/usr/lib/

	$(INSTALL_DIR) $(1)/usr/bin/nemea
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/nemea/trap_stats $(1)/usr/bin/nemea/
endef

define Package/nemea-pytrap
  $(call Package/nemea-framework)
  TITLE:=NEMEA pytrap
  DEPENDS:=+python3 +nemea-framework +libpthread
endef

define Package/nemea-pytrap/description
Python API of libtrap and UniRec
endef

define Package/nemea-pytrap/install
	cd $(PKG_BUILD_DIR)/pytrap; \
	$(HOST_PYTHON3_BIN) ./setup.py install --prefix="/usr" --root=$(1)
endef

define BuildPytrap
	cd $(PKG_BUILD_DIR)/pytrap; \
	CC="$(TARGET_CC)" \
	CCSHARED="$(TARGET_CC) $(FPIC)" \
	CXX="$(TARGET_CXX)" \
	LD="$(TARGET_CC)" \
	LDFLAGS="$(TARGET_LDFLAGS) -L$(PKG_BUILD_DIR)/libtrap/src/.libs/ -L$(PKG_BUILD_DIR)/unirec/.libs/" \
	LDSHARED="$(TARGET_CC) -shared" \
	CFLAGS="$(TARGET_CFLAGS)" \
	CPPFLAGS="$(TARGET_CPPFLAGS) -I$(PYTHON3_INC_DIR) -I$(PKG_BUILD_DIR)/libtrap/include/ -I$(PKG_BUILD_DIR)" \
	$(HOST_PYTHON3_BIN) ./setup.py build
endef

define Build/Compile
	$(call Build/Compile/Default)
	$(call BuildPytrap)
endef

$(eval $(call BuildPackage,nemea-framework,+libpthread,+librt))

$(eval $(call BuildPackage,nemea-pytrap,+libpthread,+libc))

