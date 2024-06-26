#
# Copyright (C) 2007-2015 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=mac80211

PKG_VERSION:=6.5-rc3
PKG_RELEASE:=3
# PKG_SOURCE_URL:=@KERNEL/linux/kernel/projects/backports/stable/v5.15.58/
PKG_SOURCE_URL:=http://mirror2.openwrt.org/sources/
#PKG_HASH:=5d39aca7e34c33cb9b3e366117b2e86841b7bdd37933679d6b1e61be6b150648
PKG_HASH:=90550338f957db53cee014604b95349a1c571cbadeedbb0625139a983f7fd224

PKG_SOURCE:=backports-$(PKG_VERSION).tar.xz
PKG_BUILD_DIR:=$(KERNEL_BUILD_DIR)/backports-$(PKG_VERSION)
PKG_BUILD_PARALLEL:=1

MKHASH ?= $(STAGING_DIR_HOST)/bin/mkhash
PKG_BUILD_ID:=$(shell date | $(MKHASH) md5)
PKG_MAINTAINER:=Felix Fietkau <nbd@nbd.name>

PKG_DRIVERS = \
	mac80211-hwsim \
	mt7601u \
	rsi91x rsi91x-usb rsi91x-sdio\
	wlcore wl12xx wl18xx

PKG_CONFIG_DEPENDS:= \
	CONFIG_PACKAGE_kmod-mac80211 \
	CONFIG_PACKAGE_CFG80211_TESTMODE \
	CONFIG_PACKAGE_MAC80211_MESSAGE_TRACING \
	CONFIG_PACKAGE_MAC80211_DEBUGFS \
	CONFIG_PACKAGE_MAC80211_MESH \
	CONFIG_PACKAGE_MAC80211_TRACING \
	CONFIG_PACKAGE_IWLWIFI_DEBUG \
	CONFIG_PACKAGE_IWLWIFI_DEBUGFS \
	CONFIG_PACKAGE_RTLWIFI_DEBUG \

include $(INCLUDE_DIR)/package.mk

WMENU:=Wireless Drivers

define KernelPackage/mac80211/Default
  SUBMENU:=$(WMENU)
  URL:=https://wireless.wiki.kernel.org/
  MAINTAINER:=Felix Fietkau <nbd@nbd.name>
endef

config_package=$(if $(CONFIG_PACKAGE_kmod-$(1)),m)

ifneq (,$(findstring ipq,$(CONFIG_TARGET_BOARD)))
	QCOM:=y
endif

config-y:= \
	WLAN \
	NL80211_TESTMODE \
	CFG80211_CERTIFICATION_ONUS \
	MAC80211_RC_MINSTREL \
	MAC80211_RC_MINSTREL_HT \
	MAC80211_RC_MINSTREL_VHT \
	MAC80211_RC_DEFAULT_MINSTREL \
	WLAN_VENDOR_ATH
ifneq ($(QCOM),y)
config-y += \
	WLAN_VENDOR_ADMTEK \
	WLAN_VENDOR_ATMEL \
	WLAN_VENDOR_BROADCOM \
	WLAN_VENDOR_CISCO \
	WLAN_VENDOR_INTEL \
	WLAN_VENDOR_INTERSIL \
	WLAN_VENDOR_MARVELL \
	WLAN_VENDOR_MEDIATEK \
	WLAN_VENDOR_RALINK \
	WLAN_VENDOR_REALTEK \
	WLAN_VENDOR_RSI \
	WLAN_VENDOR_ST \
	WLAN_VENDOR_TI \
	WLAN_VENDOR_ZYDAS
endif

config-$(call config_package,cfg80211) += CFG80211
config-$(CONFIG_PACKAGE_CFG80211_TESTMODE) += NL80211_TESTMODE

config-$(call config_package,mac80211) += MAC80211
config-$(CONFIG_PACKAGE_MAC80211_MESH) += MAC80211_MESH

include ath.mk
ifneq ($(QCOM),y)
  include broadcom.mk
  include intel.mk
  include marvell.mk
  include ralink.mk
  include realtek.mk
endif

PKG_CONFIG_DEPENDS += \
	$(patsubst %,CONFIG_PACKAGE_kmod-%,$(PKG_DRIVERS))

define KernelPackage/cfg80211
  $(call KernelPackage/mac80211/Default)
  TITLE:=cfg80211 - wireless configuration API
  DEPENDS+= +iw +iwinfo +wireless-regdb +USE_RFKILL:kmod-rfkill
  ABI_VERSION:=$(PKG_VERSION)-$(PKG_RELEASE)
  FILES:= \
	$(PKG_BUILD_DIR)/compat/compat.ko \
	$(PKG_BUILD_DIR)/net/wireless/cfg80211.ko
endef

define KernelPackage/cfg80211/description
cfg80211 is the Linux wireless LAN (802.11) configuration API.
endef

define KernelPackage/cfg80211/config
  if PACKAGE_kmod-cfg80211

	config PACKAGE_CFG80211_TESTMODE
		bool "Enable testmode command support"
		default n
		help
		  This is typically used for tests and calibration during
		  manufacturing, or vendor specific debugging features

  endif
endef


define KernelPackage/mac80211
  $(call KernelPackage/mac80211/Default)
  TITLE:=Linux 802.11 Wireless Networking Stack
  # +kmod-crypto-cmac is a runtime only dependency of net/mac80211/aes_cmac.c
  DEPENDS+= @TARGET_ipq95xx||TARGET_ipq53xx||TARGET_ipq50xx||TARGET_ipq60xx||TARGET_ipq807x||TARGET_ipq806x||TARGET_ipq||TARGET_ar71xx \
	+kmod-cfg80211 +hostapd-common \
	+kmod-qca-nss-ppe \
	+kmod-qca-nss-ppe-vp \
	+kmod-qca-nss-ppe-ds \
	+kmod-bonding
  PKG_BUILD_DEPENDS+= PACKAGE_kmod-qca-nss-drv:kmod-qca-nss-drv
  PKG_BUILD_DEPENDS+= PACKAGE_kmod-qca-nss-ppe:kmod-qca-nss-ppe
  PKG_BUILD_DEPENDS+= PACKAGE_kmod-qca-nss-ppe-vp:kmod-qca-nss-ppe-vp
  PKG_BUILD_DEPENDS+= PACKAGE_kmod-qca-nss-ppe-ds:kmod-qca-nss-ppe-ds
  PKG_BUILD_DEPENDS+= PACKAGE_kmod-bonding:kmod-bonding
  KCONFIG:=\
	CONFIG_AVERAGE=y
  FILES:= $(PKG_BUILD_DIR)/net/mac80211/mac80211.ko
  MENU:=1
endef

define KernelPackage/mac80211/config
  if PACKAGE_kmod-mac80211

    if PACKAGE_kmod-qca-nss-drv
	config PACKAGE_MAC80211_NSS_SUPPORT
		bool "Enable NSS support for IPQ platform"
		default y
		help
		  This option enables support for NSS in QCA boards.
    endif

    if PACKAGE_kmod-qca-nss-ppe-vp
	config PACKAGE_MAC80211_PPE_SUPPORT
		bool "Enable PPE support for IPQ platform"
		default y
		help
		  This option enables support for PPE in QCA boards
    endif

    if PACKAGE_kmod-qca-nss-ppe-ds
       config PACKAGE_MAC80211_DS_SUPPORT
               bool "Enable DS support for IPQ platform"
               default y
               help
                 This option enables support for ds netdev in QCA boards.
     endif

    if PACKAGE_kmod-qca-nss-sfe
	config PACKAGE_MAC80211_SFE_SUPPORT
		bool "Enable sfe support for IPQ platform"
		default y
		help
		  This option enables support for qca sfe in QCA boards
     endif

	config PACKAGE_MAC80211_DEBUGFS
		bool "Export mac80211 internals in DebugFS"
		select KERNEL_DEBUG_FS
		default y
		help
		  Select this to see extensive information about
		  the internal state of mac80211 in debugfs.

	config PACKAGE_MAC80211_TRACING
		bool "Enable tracing (mac80211 and supported drivers)"
		select KERNEL_FTRACE
		select KERNEL_ENABLE_DEFAULT_TRACERS
		default n
		help
		  Select this to enable tracing of mac80211 and
		  related wifi drivers (using trace-cmd).

	config PACKAGE_MAC80211_MESH
		bool "Enable 802.11s mesh support"
		default y

	config PACKAGE_MAC80211_MESSAGE_TRACING
		bool "Enable mac80211 message tracing"
		default n
		help
		  Select this option to enable tracing.

	config PACKAGE_MAC80211_DEBUG_MENU
		bool "Select mac80211 debugging features"
		default n
		help
		  This option collects various mac80211 debug settings.

	menu "mac80211 debugging features"
		depends on PACKAGE_MAC80211_DEBUG_MENU

	config PACKAGE_MAC80211_VERBOSE_DEBUG
		bool "Verbose debugging output"
		default n
		help
		  Selecting this option causes mac80211 to print out
		  many debugging messages.

	config PACKAGE_MAC80211_MLME_DEBUG
		bool "Verbose managed MLME output"
		default n
		help
		  Selecting this option causes mac80211 to print out
		  debugging messages for the managed-mode MLME.

	config PACKAGE_MAC80211_STA_DEBUG
		bool "Verbose station debugging"
		default n
		help
		  Selecting this option causes mac80211 to print out
		  debugging messages for station addition/removal.

	config PACKAGE_MAC80211_HT_DEBUG
		bool "Verbose HT debugging"
		default n
		help
		  This option enables 802.11n High Throughput features
		  debug tracing output.

	config PACKAGE_MAC80211_IBSS_DEBUG
		bool "Verbose IBSS debugging"
		default n
		help
		  Selecting this option causes mac80211 to print out
		  very verbose IBSS debugging messages.

	config PACKAGE_MAC80211_PS_DEBUG
		bool "Verbose powersave mode debugging"
		default n
		help
		  Selecting this option causes mac80211 to print out very
		  verbose power save mode debugging messages (when mac80211
		  is an AP and has power saving stations.)

        endmenu

  endif
endef

define KernelPackage/mac80211/description
Generic IEEE 802.11 Networking Stack (mac80211)
endef

define KernelPackage/mac80211-hwsim
  $(call KernelPackage/mac80211/Default)
  TITLE:=mac80211 HW simulation device
  DEPENDS+= +kmod-mac80211 +@DRIVER_11AX_SUPPORT +@DRIVER_11AC_SUPPORT
  FILES:=$(PKG_BUILD_DIR)/drivers/net/wireless/mac80211_hwsim.ko
  AUTOLOAD:=$(call AutoProbe,mac80211_hwsim)
endef


define KernelPackage/mt7601u
  $(call KernelPackage/mac80211/Default)
  TITLE:=MT7601U-based USB dongles Wireless Driver
  DEPENDS+= +kmod-mac80211 @USB_SUPPORT +kmod-usb-core +mt7601u-firmware
  FILES:=$(PKG_BUILD_DIR)/drivers/net/wireless/mediatek/mt7601u/mt7601u.ko
  AUTOLOAD:=$(call AutoProbe,mt7601u)
endef

define KernelPackage/rsi91x
  $(call KernelPackage/mac80211/Default)
  TITLE:=Redpine Signals Inc 91x WLAN driver support
  DEPENDS+= +kmod-mac80211 +rs9113-firmware
  FILES:=$(PKG_BUILD_DIR)/drivers/net/wireless/rsi/rsi_91x.ko
endef

define KernelPackage/rsi91x-usb
  $(call KernelPackage/mac80211/Default)
  TITLE:=Redpine Signals USB bus support
  DEPENDS+=@USB_SUPPORT +kmod-usb-core +kmod-mac80211 +kmod-rsi91x +rs9113-firmware
  FILES:=$(PKG_BUILD_DIR)/drivers/net/wireless/rsi/rsi_usb.ko
  AUTOLOAD:=$(call AutoProbe,rsi_usb)
endef

define KernelPackage/rsi91x-sdio
  $(call KernelPackage/mac80211/Default)
  TITLE:=Redpine Signals SDIO bus support
  DEPENDS+= +kmod-mac80211 +kmod-mmc +kmod-rsi91x +rs9113-firmware
  FILES:=$(PKG_BUILD_DIR)/drivers/net/wireless/rsi/rsi_sdio.ko
  AUTOLOAD:=$(call AutoProbe,rsi_sdio)
endef


define KernelPackage/wlcore
  $(call KernelPackage/mac80211/Default)
  TITLE:=TI common driver part
  DEPENDS+= +kmod-mmc +kmod-mac80211
  FILES:= \
	$(PKG_BUILD_DIR)/drivers/net/wireless/ti/wlcore/wlcore.ko \
	$(PKG_BUILD_DIR)/drivers/net/wireless/ti/wlcore/wlcore_sdio.ko
  AUTOLOAD:=$(call AutoProbe,wlcore wlcore_sdio)
endef

define KernelPackage/wlcore/description
 This module contains some common parts needed by TI Wireless drivers.
endef

define KernelPackage/wl12xx
  $(call KernelPackage/mac80211/Default)
  TITLE:=Driver for TI WL12xx
  URL:=https://wireless.wiki.kernel.org/en/users/drivers/wl12xx
  DEPENDS+= +kmod-wlcore +wl12xx-firmware
  FILES:=$(PKG_BUILD_DIR)/drivers/net/wireless/ti/wl12xx/wl12xx.ko
  AUTOLOAD:=$(call AutoProbe,wl12xx)
endef

define KernelPackage/wl12xx/description
 Kernel modules for TI WL12xx
endef

define KernelPackage/wl18xx
  $(call KernelPackage/mac80211/Default)
  TITLE:=Driver for TI WL18xx
  URL:=https://wireless.wiki.kernel.org/en/users/drivers/wl18xx
  DEPENDS+= +kmod-wlcore +wl18xx-firmware
  FILES:=$(PKG_BUILD_DIR)/drivers/net/wireless/ti/wl18xx/wl18xx.ko
  AUTOLOAD:=$(call AutoProbe,wl18xx)
endef

define KernelPackage/wl18xx/description
 Kernel modules for TI WL18xx
endef


ifdef CONFIG_PACKAGE_MAC80211_DEBUGFS
  config-y += \
	CFG80211_DEBUGFS \
	MAC80211_DEBUGFS
endif

ifdef CONFIG_PACKAGE_MAC80211_TRACING
  config-y += \
	IWLWIFI_DEVICE_TRACING
endif

config-$(CONFIG_PACKAGE_MAC80211_NSS_SUPPORT) += MAC80211_NSS_SUPPORT
config-$(CONFIG_PACKAGE_MAC80211_PPE_SUPPORT) += MAC80211_PPE_SUPPORT
config-$(CONFIG_PACKAGE_MAC80211_DS_SUPPORT) += MAC80211_BONDED_SUPPORT ATH12K_PPE_DS_SUPPORT ATH12K_BONDED_DS_SUPPORT
config-$(CONFIG_PACKAGE_MAC80211_SFE_SUPPORT) += MAC80211_SFE_SUPPORT
config-$(CONFIG_PACKAGE_MAC80211_MESSAGE_TRACING) += MAC80211_MESSAGE_TRACING ATH10K_TRACING ATH11K_TRACING ATH12K_TRACING
config-$(CONFIG_PACKAGE_MAC80211_DEBUG_MENU) += MAC80211_DEBUG_MENU
config-$(CONFIG_PACKAGE_MAC80211_VERBOSE_DEBUG) += MAC80211_VERBOSE_DEBUG
config-$(CONFIG_PACKAGE_MAC80211_MLME_DEBUG) += MAC80211_MLME_DEBUG
config-$(CONFIG_PACKAGE_MAC80211_STA_DEBUG) += MAC80211_STA_DEBUG
config-$(CONFIG_PACKAGE_MAC80211_HT_DEBUG) += MAC80211_HT_DEBUG
config-$(CONFIG_PACKAGE_MAC80211_IBSS_DEBUG) += MAC80211_IBSS_DEBUG
config-$(CONFIG_PACKAGE_MAC80211_PS_DEBUG) += MAC80211_PS_DEBUG


config-$(call config_package,mac80211-hwsim) += MAC80211_HWSIM
ifneq ($(QCOM),y)
  config-$(call config_package,mt7601u) += MT7601U
  config-y += WL_MEDIATEK

  config-$(call config_package,wlcore) += WLCORE WLCORE_SDIO
  config-$(call config_package,wl12xx) += WL12XX
  config-$(call config_package,wl18xx) += WL18XX
  config-y += WL_TI WILINK_PLATFORM_DATA
  config-$(call config_package,rsi91x) += RSI_91X
  config-$(call config_package,rsi91x-usb) += RSI_USB
  config-$(call config_package,rsi91x-sdio) += RSI_SDIO
  config-$(CONFIG_LEDS_TRIGGERS) += MAC80211_LEDS
endif

C_DEFINES=

ifeq ($(BUILD_VARIANT),smallbuffers)
	C_DEFINES+= -DCONFIG_ATH10K_SMALLBUFFERS
endif

MAKE_OPTS:= \
	$(subst -C $(LINUX_DIR),-C "$(PKG_BUILD_DIR)",$(KERNEL_MAKEOPTS)) \
	EXTRA_CFLAGS="-I$(PKG_BUILD_DIR)/include -I$(STAGING_DIR)/usr/include/qca-nss-drv -I$(STAGING_DIR)/usr/include/qca-nss-ppe -I$(STAGING_DIR)/usr/include/qca-nss-clients -I$(TOPDIR)/qca/src/qca-wifi/telemetry_agent/inc/ -Wall -Werror -Wno-incompatible-pointer-types -Wno-discarded-qualifiers -Wno-int-conversion" \
	KLIB_BUILD="$(LINUX_DIR)" \
	MODPROBE=true \
	KLIB=$(TARGET_MODULES_DIR) \
	KERNEL_SUBLEVEL=$(lastword $(subst ., ,$(KERNEL_PATCHVER))) \
	KBUILD_LDFLAGS_MODULE_PREREQ=

define ConfigVars
$(subst $(space),,$(foreach opt,$(config-$(1)),CPTCFG_$(opt)=$(1)
))
endef

define mac80211_config
$(call ConfigVars,m)$(call ConfigVars,y)
endef
$(eval $(call shexport,mac80211_config))

define Build/Prepare
	rm -rf $(PKG_BUILD_DIR)
	mkdir -p $(PKG_BUILD_DIR)
	$(PKG_UNPACK)
	$(Build/Patch)
	rm -rf \
		$(PKG_BUILD_DIR)/include/linux/ssb \
		$(PKG_BUILD_DIR)/include/linux/bcma \
		$(PKG_BUILD_DIR)/include/net/bluetooth

	rm -f \
		$(PKG_BUILD_DIR)/include/linux/cordic.h \
		$(PKG_BUILD_DIR)/include/linux/crc8.h \
		$(PKG_BUILD_DIR)/include/linux/eeprom_93cx6.h \
		$(PKG_BUILD_DIR)/include/linux/wl12xx.h \
		$(PKG_BUILD_DIR)/include/linux/mhi.h \
		$(PKG_BUILD_DIR)/include/net/ieee80211.h \
		$(PKG_BUILD_DIR)/backport-include/linux/bcm47xx_nvram.h

	echo 'compat-wireless-$(PKG_VERSION)-$(PKG_RELEASE)-$(REVISION)' > $(PKG_BUILD_DIR)/compat_version
endef

ifneq ($(CONFIG_PACKAGE_kmod-cfg80211),)
 define Build/Compile/kmod
	rm -rf $(PKG_BUILD_DIR)/modules
	+$(MAKE) $(PKG_JOBS) $(MAKE_OPTS) modules
 endef
endif

EXTERNAL_PATCH_DIR:=$(TOPDIR)/openwrt-patches/package/kernel/mac80211/patches

define Build/Patch
	$(if $(QUILT),rm -rf $(PKG_BUILD_DIR)/patches; mkdir -p $(PKG_BUILD_DIR)/patches)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/build,build/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/subsys,subsys/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath,ath/)
#	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath5k,ath5k/)
#	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath9k,ath9k/)
#	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath10k,ath10k/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath11k,ath11k/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath12k,ath12k/)
ifneq ($(QCOM),y)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/rt2x00,rt2x00/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/mt7601u,mt7601u/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/mwl,mwl/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/brcm,brcm/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/rtl,rtl/)
endif
	$(call PatchDir,$(PKG_BUILD_DIR),$(EXTERNAL_PATCH_DIR)/subsys,subsys/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(EXTERNAL_PATCH_DIR)/ath11k,ath11k/)
	$(call PatchDir,$(PKG_BUILD_DIR),$(EXTERNAL_PATCH_DIR)/ath12k,ath12k/)
	$(if $(QUILT),touch $(PKG_BUILD_DIR)/.quilt_used)
endef

define Quilt/Refresh/Package
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/build,build/)
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/subsys,subsys/)
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath,ath/)
#	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath5k,ath5k/)
#	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath9k,ath9k/)
#	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath10k,ath10k/)
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath11k,ath11k/)
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/ath12k,ath12k/)
ifneq ($(QCOM),y)
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/rt2x00,rt2x00/)
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/mt7601u,mt7601u/)
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/mwl,mwl/)
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/brcm,brcm/)
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/rtl,rtl/)
endif
endef

define Build/Compile
	$(SH_FUNC) var2file "$(call shvar,mac80211_config)" $(PKG_BUILD_DIR)/.config
	$(MAKE) $(MAKE_OPTS) allnoconfig
	$(call Build/Compile/kmod)
endef

define Build/InstallDev
	mkdir -p \
		$(1)/usr/include/mac80211 \
		$(1)/usr/include/mac80211-backport \
		$(1)/usr/include/mac80211/ath \
		$(1)/usr/include/net/mac80211
	$(CP) $(PKG_BUILD_DIR)/net/mac80211/*.h $(PKG_BUILD_DIR)/include/* $(1)/usr/include/mac80211/
	$(CP) $(PKG_BUILD_DIR)/backport-include/* $(1)/usr/include/mac80211-backport/
	$(CP) $(PKG_BUILD_DIR)/net/mac80211/rate.h $(1)/usr/include/net/mac80211/
	$(CP) $(PKG_BUILD_DIR)/drivers/net/wireless/ath/*.h $(1)/usr/include/mac80211/ath/
	rm -f $(1)/usr/include/mac80211-backport/linux/module.h
endef

EXTERNAL_FILE_DIR:=$(TOPDIR)/openwrt-patches/package/kernel/mac80211

define KernelPackage/cfg80211/install
	$(INSTALL_DIR) $(1)/lib/wifi $(1)/etc/hotplug.d/ieee80211
	$(INSTALL_DIR) $(1)/lib/wifi $(1)/etc/hotplug.d/devcoredump
	$(INSTALL_DIR) $(1)/lib/wifi $(1)/lib/netifd/wireless
	$(INSTALL_DIR) $(1)/lib/wifi/sawf
	$(INSTALL_DIR) $(1)/lib/wifi/sawf/telemetry
	$(INSTALL_DATA) ./files/lib/wifi/mac80211.sh $(1)/lib/wifi
	$(INSTALL_DATA) $(EXTERNAL_FILE_DIR)/files/lib/wifi/def_service_classes.json $(1)/lib/wifi/sawf/
	$(INSTALL_DATA) $(EXTERNAL_FILE_DIR)/files/lib/wifi/service_classes.json $(1)/lib/wifi/sawf/
	$(INSTALL_DATA) $(EXTERNAL_FILE_DIR)/files/lib/wifi/telemetry/config.json $(1)/lib/wifi/sawf/telemetry/
	$(INSTALL_DATA) $(EXTERNAL_FILE_DIR)/files/lib/wifi/telemetry/sla.json $(1)/lib/wifi/sawf/telemetry/
	$(INSTALL_DATA) $(EXTERNAL_FILE_DIR)/files/lib/wifi/telemetry/sla_detect.json $(1)/lib/wifi/sawf/telemetry/
	$(INSTALL_BIN) ./files/lib/netifd/wireless/mac80211.sh $(1)/lib/netifd/wireless
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/lib/performance.sh $(1)/lib
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/lib/smp_affinity_settings.sh $(1)/lib
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/lib/boost_performance.sh $(1)/lib
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/lib/ds_enable.sh $(1)/lib
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/lib/apsta_mode.sh $(1)/lib
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/coredump.sh $(1)/etc/hotplug.d/devcoredump
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/lib/wifi-config.sh $(1)/etc/hotplug.d/ieee80211/01-wifi-detect
endef

define KernelPackage/ath/install
	$(INSTALL_DIR) $(STAGING_DIR)/usr/include/
	$(CP) $(PKG_BUILD_DIR)/include/ath/ath_sawf.h $(STAGING_DIR)/usr/include/
	$(CP) $(PKG_BUILD_DIR)/include/ath/ath_fse.h $(STAGING_DIR)/usr/include/
	$(CP) $(PKG_BUILD_DIR)/include/ath/ath_dp_accel_cfg.h $(STAGING_DIR)/usr/include/
endef

define KernelPackage/ath11k/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/etc/init.d/ath11k_nss_enable.sh $(1)/etc/init.d
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/etc/init.d/ath11k_uboot_mod_params.sh $(1)/etc/init.d
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/etc/init.d/qca-nss-pbuf $(1)/etc/init.d
endef

define KernelPackage/ath12k/install
	$(call KernelPackage/ath/install)
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(EXTERNAL_FILE_DIR)/files/etc/init.d/ath12k_dyn_dbg_enable.sh $(1)/etc/init.d
endef

$(eval $(foreach drv,$(PKG_DRIVERS),$(call KernelPackage,$(drv))))
$(eval $(call KernelPackage,cfg80211))
$(eval $(call KernelPackage,mac80211))
