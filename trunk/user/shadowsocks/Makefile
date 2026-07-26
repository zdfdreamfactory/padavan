THISDIR = $(shell pwd)

SRC_SS_NAME=shadowsocks-libev-3.3.6
#https://github.com/shadowsocks/shadowsocks-libev.git

SRC_SSR_NAME=shadowsocksr-libev-master
ARCHIVE_SSR_NAME=shadowsocksr-libev-master-e2373d7
#https://github.com/shadowsocksr-backup/shadowsocksr-libev.git

# Build directories
BUILD_SS_DIR=$(SRC_SS_NAME)/build
BUILD_SSR_DIR=$(SRC_SSR_NAME)/build


all: extract_test config_test
ifeq ($(CONFIG_FIRMWARE_INCLUDE_SHADOWSOCKS),y)
	$(MAKE) -j$(HOST_NCPU) -C $(SRC_SSR_NAME)
endif
	$(MAKE) -j$(HOST_NCPU) -C $(SRC_SS_NAME)
ifeq ($(CONFIG_FIRMWARE_INCLUDE_SHADOWSOCKS),y)
extract_test: extract_ssr_test extract_ss_test
else
extract_test: extract_ss_test
endif

extract_ssr_test:
	( if [ ! -d $(SRC_SSR_NAME) ]; then \
		unzip -qq $(ARCHIVE_SSR_NAME).zip ; \
		cd $(SRC_SSR_NAME); \
		patch -p1 < ../shadowsocksr-libev-master-e2373d7.diff; \
	fi )

extract_ss_test:
	( if [ ! -d $(SRC_SS_NAME) ]; then \
		tar xf $(SRC_SS_NAME).tar.gz ; \
	fi )

ifeq ($(CONFIG_FIRMWARE_INCLUDE_SHADOWSOCKS),y)
config_test: config_ssr config_ss
else
config_test: config_ss
endif

config_ss:
	( if [ -f ./ss_config_done ]; then \
		echo "the same configuration"; \
	else \
		make configure_ss && touch ss_config_done; \
	fi )

config_ssr:
	( if [ -f ./ssr_config_done ]; then \
		echo "the same configuration"; \
	else \
		make configure_ssr && touch ssr_config_done; \
	fi )

configure_ssr:
	( cd $(SRC_SSR_NAME) ; \
	aclocal -I m4; autoreconf -fi; automake -a --copy; \
	ac_cv_prog_PCRE_CONFIG="$(STAGEDIR)/bin/pcre-config" \
	./configure \
		--prefix=/usr \
		--disable-documentation \
		--disable-ssp \
		--disable-assert \
		--enable-system-shared-lib \
		--with-crypto-library=mbedtls \
		--host=$(HOST_TARGET) \
		--build=$(HOST_BUILD) ; \
	)

configure_ss:
	( cd $(SRC_SS_NAME) ; \
	echo "Configuring shadowsocks-libev-3.3.6 with CMake..."; \
	mkdir -p $(BUILD_SS_DIR) ; \
	cd $(BUILD_SS_DIR) ; \
	cmake .. \
		-DCMAKE_BUILD_TYPE=Release \
		-DCMAKE_INSTALL_PREFIX=/usr \
		-DCMAKE_TOOLCHAIN_FILE=$(CONFIG_CMAKE_TOOLCHAIN_FILE) \
		-DWITH_DOC_MAN=OFF \
		-DBUILD_TESTING=OFF \
		-DWITH_EMBEDDED_SRC=ON \
		-DENABLE_CONNMARKTOS=OFF \
		-DENABLE_NFTABLES=OFF \
		2>&1 | grep -E "^(--|  |ERROR|WARNING)" || true ; \
	if [ ! -f CMakeCache.txt ]; then \
		echo "ERROR: CMake configuration failed"; \
		exit 1; \
	fi; \
	)

clean: clean_ssr clean_ss
	rm -f ssr_config_done ss_config_done

clean_ssr:
	( if [ -f $(SRC_SSR_NAME)/Makefile ] ; then \
		$(MAKE) -C $(SRC_SSR_NAME) clean ; \
	fi )

clean_ss:
	( if [ -d $(BUILD_SS_DIR) ]; then \
		rm -rf $(BUILD_SS_DIR) ; \
	elif [ -f $(SRC_SS_NAME)/Makefile ]; then \
		$(MAKE) -C $(SRC_SS_NAME) distclean ; \
	fi )
	
romfs:
ifeq ($(CONFIG_FIRMWARE_INCLUDE_SHADOWSOCKS),y)
	chmod -R +x scripts/
	$(ROMFSINST) $(THISDIR)/scripts/ /usr/bin/
	$(ROMFSINST) $(THISDIR)/oversea/oversea_list.conf /etc_ro/oversea_list.conf
	$(ROMFSINST) $(THISDIR)/gfwlist/gfwlist_list.conf /etc_ro/gfwlist_list.conf
	$(ROMFSINST) /etc_ro/ss_dom.sh
	$(ROMFSINST) /etc_ro/uss_dom.sh
	$(ROMFSINST) /etc_ro/ss_ip.sh
	$(ROMFSINST) /etc_ro/ss_lan_bip.sh
	$(ROMFSINST) /etc_ro/ss_lan_gmip.sh
	$(ROMFSINST) /etc_ro/ss_lan_ip.sh
	$(ROMFSINST) /etc_ro/ss_wan_ip.sh
	$(ROMFSINST) /etc_ro/ss_dlink.sh
	$(ROMFSINST) $(THISDIR)/ss/ /etc_ro/ss/
	$(ROMFSINST) -p +x $(THISDIR)/$(SRC_SSR_NAME)/src/ss-redir /usr/bin/ssr-redir
	$(ROMFSINST) -p +x $(THISDIR)/$(SRC_SSR_NAME)/src/ss-local /usr/bin/ssr-local
	$(ROMFSINST) -p +x $(THISDIR)/$(SRC_SS_NAME)/src/ss-redir /usr/bin/ss-redir
	$(ROMFSINST) -p +x $(THISDIR)/$(SRC_SS_NAME)/src/ss-local /usr/bin/ss-local
endif
ifeq ($(CONFIG_FIRMWARE_INCLUDE_SSSERVER),y)
	$(ROMFSINST) -p +x $(THISDIR)/$(SRC_SS_NAME)/src/ss-server /usr/bin/ss-server
endif

