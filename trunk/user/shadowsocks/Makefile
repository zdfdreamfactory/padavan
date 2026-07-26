THISDIR = $(shell pwd)

SRC_SS_NAME=shadowsocks-libev-3.3.6
SRC_SSR_NAME=shadowsocksr-libev-master
ARCHIVE_SSR_NAME=shadowsocksr-libev-master-e2373d7

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
		if [ ! -f $(SRC_SS_NAME).tar.gz ]; then \
			echo "Downloading shadowsocks-libev-3.3.6 source..."; \
			curl -sL --connect-timeout 30 -O https://github.com/shadowsocks/shadowsocks-libev/releases/download/v3.3.6/shadowsocks-libev-3.3.6.tar.gz || true; \
		fi; \
		tar xzf $(SRC_SS_NAME).tar.gz ; \
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
		make do_configure_ss && touch ss_config_done; \
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

# Shadowsocks-libev 3.3.6 uses CMake (not autotools).
# Key changes from original:
#   1. cmake -S . -B build — build in-source-relative (from SRC_SS_NAME dir), explicit source=.
#   2. No CMAKE_PREFIX_PATH or CMAKE_FIND_ROOT_PATH on CLI — let toolchain file handle via env vars
#   3. Explicitly set cross compiler paths to bypass any root-path mapping issues
#   4. WITH_EMBEDDED_SRC=OFF to prevent embedded dep build failures in cross-compilation
do_configure_ss:
	( cd $(SRC_SS_NAME) ; \
	echo "Configuring shadowsocks-libev with CMake..."; \
	rm -f CMakeCache.txt ; rm -rf CMakeFiles/ ; \
	cmake -G "Unix Makefiles" . \
		-DCMAKE_SYSTEM_NAME=Linux \
		-DCMAKE_SYSTEM_PROCESSOR=mipsel \
		-DCMAKE_C_COMPILER=$(CROSS_COMPILE)gcc \
		-DCMAKE_CXX_COMPILER=$(CROSS_COMPILE)g++ \
		-DCMAKE_AR=$(CROSS_COMPILE)ar \
		-DCMAKE_NM=$(CROSS_COMPILE)nm \
		-DCMAKE_RANLIB=$(CROSS_COMPILE)ranlib \
		-DCMAKE_BUILD_TYPE=Release \
		-DCMAKE_INSTALL_PREFIX=/usr \
		-DCMAKE_SKIP_RPATH=TRUE \
		-DCMAKE_PREFIX_PATH=$(STAGEDIR) \
		-DWITH_MBEDTLS=ON \
		-DWITH_SODIUM=OFF \
		-DWITH_EVPORT=OFF \
		-DWITH_EMBEDDED_SRC=OFF \
		-DENABLE_NAT=ON \
		-DENABLE_MANPAGE=OFF \
		-DBUILD_TESTING=OFF \
		-DWITH_USR=OFF \
		2>&1 | tee ../config.log; \
	if [ ! -f CMakeCache.txt ]; then \
		echo "ERROR: CMake configuration failed!"; \
		echo "=== Last 50 lines of config log ==="; \
		tail -50 ../config.log; \
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
	( if [ -d $(SRC_SS_NAME) ]; then \
		rm -rf $(SRC_SS_NAME)/$(BUILD_SS_DIR) ; \
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
	$(ROMFSINST) -p +x $(THISDIR)/$(SRC_SS_NAME)/$(BUILD_SS_DIR)/src/ss-redir /usr/bin/ss-redir
	$(ROMFSINST) -p +x $(THISDIR)/$(SRC_SS_NAME)/$(BUILD_SS_DIR)/src/ss-local /usr/bin/ss-local
endif
ifeq ($(CONFIG_FIRMWARE_INCLUDE_SSSERVER),y)
	$(ROMFSINST) -p +x $(THISDIR)/$(SRC_SS_NAME)/$(BUILD_SS_DIR)/src/ss-server /usr/bin/ss-server
endif

