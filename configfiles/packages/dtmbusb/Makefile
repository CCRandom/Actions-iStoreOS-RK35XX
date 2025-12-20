include $(TOPDIR)/rules.mk

PKG_NAME:=dtmbusb
PKG_VERSION:=1.0
PKG_RELEASE:=1

# No source download - use local files
PKG_SOURCE:=
PKG_SOURCE_URL:=
PKG_SOURCE_PROTO:=
PKG_SOURCE_VERSION:=
PKG_MIRROR_HASH:=

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/package.mk

define KernelPackage/dtmbusb
  SUBMENU:=Video Support
  TITLE:=DtmbUSB DTMB demodulator driver
  FILES:=$(PKG_BUILD_DIR)/dtmbusb-dev.ko
  FILES+=$(PKG_BUILD_DIR)/dtmbusb-fe.ko
  FILES+=$(PKG_BUILD_DIR)/dvb-core.ko
  FILES+=$(PKG_BUILD_DIR)/dvb-usb.ko
  FILES+=$(PKG_BUILD_DIR)/dvb_usb_v2.ko
  AUTOLOAD:=$(call AutoLoad,90,dtmbusb-dev dtmbusb-fe dvb-core dvb-usb dvb_usb_v2)
endef

define KernelPackage/dtmbusb/description
  Linux DVB driver for Letv/Aiwa/CVB DtmbUSB DTMB demodulator USB dongle.
endef

define Build/Prepare
	$(call Build/Prepare/Default)
endef

define Build/Compile
	# Create linuxdvb directory structure similar to original DtmbUSB
	mkdir -p $(PKG_BUILD_DIR)/linuxdvb
	
	# Copy DVB core source files from kernel to linuxdvb directory (matching original structure)
	cp -f $(LINUX_DIR)/drivers/media/dvb-core/*.c $(PKG_BUILD_DIR)/linuxdvb/ 2>/dev/null || true
	cp -f $(LINUX_DIR)/drivers/media/usb/dvb-usb/*.c $(PKG_BUILD_DIR)/linuxdvb/ 2>/dev/null || true
	cp -f $(LINUX_DIR)/drivers/media/usb/dvb-usb-v2/*.c $(PKG_BUILD_DIR)/linuxdvb/ 2>/dev/null || true
	
	# Also copy header files if they exist in the source locations
	cp -f $(LINUX_DIR)/drivers/media/dvb-core/*.h $(PKG_BUILD_DIR)/linuxdvb/ 2>/dev/null || true
	cp -f $(LINUX_DIR)/drivers/media/usb/dvb-usb/*.h $(PKG_BUILD_DIR)/linuxdvb/ 2>/dev/null || true
	cp -f $(LINUX_DIR)/drivers/media/usb/dvb-usb-v2/*.h $(PKG_BUILD_DIR)/linuxdvb/ 2>/dev/null || true

	# Create a Makefile that closely matches original DtmbUSB Makefile
	cat > $(PKG_BUILD_DIR)/Makefile << 'EOF'
# DtmbUSB Makefile adapted for OpenWrt build system
ccflags-y += -D CONFIG_DVB_NET

# DVB core objects - match original structure
dvb-core-objs += linuxdvb/dvbdev.o linuxdvb/dmxdev.o linuxdvb/dvb_demux.o 
dvb-core-objs += linuxdvb/dvb_ca_en50221.o linuxdvb/dvb_frontend.o linuxdvb/dvb_net.o 
dvb-core-objs += linuxdvb/dvb_ringbuffer.o linuxdvb/dvb_math.o

# For older kernel versions (<= 4.10.0)
# dvb-core-objs += linuxdvb/dvb_filter.o

obj-m += dvb-core.o

# DVB USB objects - match original structure
dvb-usb-objs += linuxdvb/dvb-usb-firmware.o linuxdvb/dvb-usb-init.o linuxdvb/dvb-usb-urb.o 
dvb-usb-objs += linuxdvb/dvb-usb-i2c.o linuxdvb/dvb-usb-dvb.o linuxdvb/usb-urb.o
# Conditionally include remote support if available
dvb-usb-objs += $(if $(CONFIG_RC_CORE),linuxdvb/dvb-usb-remote.o,)

obj-m += dvb-usb.o

# DVB USB v2 objects - match original structure
dvb_usb_v2-objs := linuxdvb/dvb_usb_core.o linuxdvb/dvb_usb_urb.o linuxdvb/usb_urb.o
obj-m += dvb_usb_v2.o

# DtmbUSB objects
obj-m += dtmbusb-fe.o
obj-m += dtmbusb-dev.o

# Include paths - match original but use OpenWrt variables
ccflags-y += -I$(LINUX_DIR)/drivers/media/dvb-core
ccflags-y += -I$(LINUX_DIR)/drivers/media/usb/dvb-usb
ccflags-y += -I$(LINUX_DIR)/drivers/media/usb/dvb-usb-v2
ccflags-y += -I$(LINUX_DIR)/drivers/media/dvb-frontends
EOF

	# Build the modules using OpenWrt's kernel build system
	$(MAKE) -C "$(LINUX_DIR)" \
		ARCH="$(LINUX_KARCH)" \
		CROSS_COMPILE="$(TARGET_CROSS)" \
		M="$(PKG_BUILD_DIR)" \
		modules
endef

define KernelPackage/dtmbusb/install
	$(INSTALL_DIR) $(1)/etc/modules.d
	echo "dtmbusb-dev" > $(1)/etc/modules.d/90-dtmbusb
	echo "dtmbusb-fe" >> $(1)/etc/modules.d/90-dtmbusb
	echo "dvb-core" >> $(1)/etc/modules.d/90-dtmbusb
	echo "dvb-usb" >> $(1)/etc/modules.d/90-dtmbusb
	echo "dvb_usb_v2" >> $(1)/etc/modules.d/90-dtmbusb
endef

$(eval $(call KernelPackage,dtmbusb))
