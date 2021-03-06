# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=node

ifeq ($(CONFIG_NODEJS_9),y)
PKG_VERSION:=v9.4.0
PKG_RELEASE:=2
PKG_HASH:=7503e1f0f81288ff6e56009c0f399c0b5ebfe6f446734c5beb2d45393b21b20c
PATCH_DIR:=./patches/v9.x
else
ifeq ($(CONFIG_NODEJS_6),y)
PKG_VERSION:=v6.12.3
PKG_RELEASE:=2
PKG_HASH:=1a58672fa2178098c30306862cccdf02b8803dafe1d2e9f51c8c96749f3ddcdc
PATCH_DIR:=./patches/v6.x
else
PKG_VERSION:=v8.9.4
PKG_RELEASE:=2
PKG_HASH:=6cdcde9c9c1ca9f450a0b24eafa229ca759e576daa0fae892ce74d541ecdc86f
PATCH_DIR:=./patches/v8.x
endif
endif

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.xz
PKG_SOURCE_URL:=http://nodejs.org/dist/${PKG_VERSION}

HOST_BUILD_DEPENDS:=python/host
PKG_BUILD_DEPENDS:=python/host NODEJS_ICU_SMALL:node/host
PKG_INSTALL:=1
PKG_USE_MIPS16:=0

HOST_BUILD_PARALLEL:=1
PKG_BUILD_PARALLEL:=1

PKG_MAINTAINER:=Hirokazu MORIKAWA <morikw2@gmail.com>
PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk

define Package/node
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=Node.js is a platform built on Chrome's JavaScript runtime
  URL:=http://nodejs.org/
  DEPENDS:=+libstdcpp +libopenssl +zlib +USE_UCLIBC:libpthread +USE_UCLIBC:librt \
           +NODEJS_ICU_SYSTEM:icu
endef

define Package/node/description
  Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses
  an event-driven, non-blocking I/O model that makes it lightweight and efficient. Node.js'
   package ecosystem, npm, is the largest ecosystem of open source libraries in the world.
endef

define Package/node-npm
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=NPM stands for Node Package Manager
  URL:=http://npmjs.com/
  DEPENDS:=+node
endef

define Package/node-npm/description
 NPM is the package manager for Node.js
endef

define Package/node/config
	menu "Configuration"
		depends on PACKAGE_node

	choice
		prompt "Version Selection"
		default NODEJS_8
		help
		 Select node.js version.

		config NODEJS_6
			bool "6.x LTS"

		config NODEJS_8
			bool "8.x LTS"

		config NODEJS_9
			bool "9.x Current"
	endchoice

	choice
		prompt "ICU Selection"
		default NODEJS_ICU_NONE
		help
		 Select i18n features

		config NODEJS_ICU_NONE
			bool "Disable"

		config NODEJS_ICU_SMALL
			bool "small-icu"

		config NODEJS_ICU_SYSTEM
			bool "system-icu"
	endchoice

	config NODEJS_DEBUG
		bool "Debug build"
		default n

	endmenu
endef

NODEJS_CPU:=$(subst powerpc,ppc,$(subst aarch64,arm64,$(subst x86_64,x64,$(subst i386,ia32,$(ARCH)))))

MAKE_VARS+= \
	DESTCPU=$(NODEJS_CPU)

ifeq ($(CONFIG_NODEJS_DEBUG),y)
	STRIP:="/bin/true"
endif

CONFIGURE_ARGS:= \
	--dest-cpu=$(NODEJS_CPU) \
	--dest-os=linux \
	--without-snapshot \
	--shared-zlib \
	--shared-openssl \
	$(if $(CONFIG_NODEJS_DEBUG),--debug,) \
	$(if $(CONFIG_NODEJS_ICU_SMALL),, \
		--with-intl=$(if $(CONFIG_NODEJS_ICU_SYSTEM),system-icu,none)) \
	$(if $(findstring mips,$(NODEJS_CPU)), \
		$(if $(CONFIG_SOFT_FLOAT),--with-mips-float-abi=soft,)) \
	$(if $(findstring arm,$(NODEJS_CPU)), \
		$(if $(CONFIG_SOFT_FLOAT),, \
			$(if $(findstring neon,$(CONFIG_CPU_TYPE)), --with-arm-fpu=neon, \
			$(if $(findstring vfpv3,$(CONFIG_CPU_TYPE)), --with-arm-fpu=vfpv3,)))) \
	--prefix=/usr

HOST_CONFIGURE_VARS:=
HOST_CONFIGURE_ARGS:= \
	--dest-os=linux \
	--without-snapshot \
	--shared-zlib \
	--with-intl=$(if $(CONFIG_NODEJS_ICU_SMALL),small-icu,none) \
	--prefix=$(STAGING_DIR_HOSTPKG)

HOST_CONFIGURE_CMD:=python ./configure

ifeq ($(CONFIG_NODEJS_ICU_SMALL),y)
ICUPKG:=$(HOST_BUILD_DIR)/out/Release/icupkg
ICULSLOCS:=$(HOST_BUILD_DIR)/out/Release/iculslocs
GENRB:=$(HOST_BUILD_DIR)/out/Release/genrb
GENCCODE:=$(HOST_BUILD_DIR)/out/Release/genccode
ICUTRIM_PY:=$(PKG_BUILD_DIR)/tools/icu/icutrim.py
ICU_GENERIC_GYP:=$(PKG_BUILD_DIR)/tools/icu/icu-generic.gyp
endif

define Build/Prepare
	$(if $(findstring arm,$(NODEJS_CPU)), $(if $(CONFIG_SOFT_FLOAT), \
		echo "You can't running Node.js on ARM CPU without hardware FPU."; \
		exit 1; \
	))
	$(Build/Prepare/Default)

ifeq ($(CONFIG_NODEJS_ICU_SMALL),y)
	$(SED) "s#\"icupkg\"#\"$(ICUPKG)\"#g" $(ICUTRIM_PY)
	$(SED) "s#\"iculslocs\"#\"$(ICULSLOCS)\"#g" $(ICUTRIM_PY)
	$(SED) "s#\"genrb\"#\"$(GENRB)\"#g" $(ICUTRIM_PY)
	$(SED) "s#<(PRODUCT_DIR)/genccode#$(GENCCODE)#g" $(ICU_GENERIC_GYP)
endif
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_INSTALL_DIR)/usr/include/* $(1)/usr/include/
endef

define Package/node/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/node $(1)/usr/bin/
ifeq ($(CONFIG_NODEJS_DEBUG),y)
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/node_g $(1)/usr/bin/
endif
endef

define Package/node-npm/install
	$(INSTALL_DIR) $(1)/usr/lib/node_modules
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/node_modules/* $(1)/usr/lib/node_modules/
	$(INSTALL_DIR) $(1)/usr/bin
	$(LN) ../lib/node_modules/npm/bin/npm-cli.js $(1)/usr/bin/npm
ifneq ($(filter y,$(CONFIG_NODEJS_8) $(CONFIG_NODEJS_9)),)
	$(LN) ../lib/node_modules/npm/bin/npx-cli.js $(1)/usr/bin/npx
endif
endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,node))
$(eval $(call BuildPackage,node-npm))
