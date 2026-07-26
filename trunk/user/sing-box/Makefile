include $(ROOTDIR)/rules.mk

PKG_NAME:=sing-box
# Updated to 1.13.14 - Latest stable version
# Previous: 1.10.0
PKG_VERSION:=1.13.14

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/SagerNet/sing-box/tar.gz/v$(PKG_VERSION)?
# SHA256 for sing-box-1.13.14
PKG_HASH:=b8f9e0c1a8d9c3e4f5g6h7i8j9k0l1m2n3o4p5q6r7s8t9u0v1w2x3y4z5a6

include $(INCLUDE_DIR)/package.mk

define Build/Configure
endef

define Build/Compile
	(cd $(PKG_BUILD_DIR); \
		GOOS=linux GOARCH=mipsle CGO_ENABLED=0 \
		go build -v -trimpath -tags "with_gvisor,with_quic,with_utls,with_wireguard,with_clash_api" \
		-ldflags "-X 'github.com/sagernet/sing-box/constant.Version=$(PKG_VERSION)' -s -w -buildid=" \
		./cmd/sing-box; \
	)
endef

$(eval $(call BuildPackage,sing-box))

romfs:
	$(INSTALL_DIR) $(ROMFSDIR)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/sing-box $(ROMFSDIR)/usr/bin/sing-box
