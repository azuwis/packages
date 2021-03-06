include $(TOPDIR)/rules.mk

PKG_NAME:=docker-ce
PKG_VERSION:=18.09.6
PKG_RELEASE:=1
PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=components/cli/LICENSE components/engine/LICENSE

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/docker/docker-ce/tar.gz/v$(PKG_VERSION)?
PKG_HASH:=e8d2dd41e09e838e9043d4a5cf8433d8860afa20a611025621f7817b7ab16012
PKG_SOURCE_VERSION:=481bc77156

PKG_MAINTAINER:=Gerard Ryan <G.M0N3Y.2503@gmail.com>

PKG_BUILD_DEPENDS:=golang/host
PKG_BUILD_PARALLEL:=1

GO_PKG:=github.com/docker

include $(INCLUDE_DIR)/package.mk
include ../../lang/golang/golang-package.mk

define Package/docker-ce/config
  source "$(SOURCE)/Config.in"
endef

define Package/docker-ce
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=Docker Community Edition
  URL:=https://www.docker.com/
  DEPENDS:=$(GO_ARCH_DEPENDS) @TARGET_x86_64 +btrfs-progs +libdevmapper +containerd +cgroupfs-mount +ca-certificates +iptables-mod-extra +DOCKER_SECCOMP:libseccomp \
           +kmod-ikconfig +kmod-veth +kmod-br-netfilter +kmod-nf-ipvs
  USERID:=docker:docker
  MENU:=1
endef

define Package/docker-ce/description
  Docker Engine is used by millions enables containerized applications
  to run anywhere consistently on any infrastructure.
endef

define Build/Configure
	# move so GoPackage/Build/Configure will get the correct path
	mv $(PKG_BUILD_DIR)/components/engine $(PKG_BUILD_DIR)/
	mv $(PKG_BUILD_DIR)/components/cli $(PKG_BUILD_DIR)/

	# docker generates files at build time so we'll just symlink for now and call GoPackage/Build/Configure later
	mkdir -p $(GO_PKG_BUILD_DIR)/bin \
			 $(GO_PKG_BUILD_DIR)/src \
			 $(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/ \
			 $(GO_PKG_CACHE_DIR) \
			 $(GO_PKG_TMP_DIR)
	$(LN) $(PKG_BUILD_DIR)/cli $(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/cli
	$(LN) $(PKG_BUILD_DIR)/engine $(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/docker
endef

ifeq ($(CONFIG_DOCKER_SECCOMP),y)
BUILDTAGS:=seccomp
else
BUILDTAGS:=
endif

define Build/Compile
	( \
		export GOPATH=$(GO_PKG_BUILD_DIR) \
			GOCACHE=$(GO_PKG_CACHE_DIR) \
			GOTMPDIR=$(GO_PKG_TMP_DIR) \
			GOROOT_FINAL=$(GO_TARGET_ROOT) \
			CC=$(TARGET_CC) \
			CXX=$(TARGET_CXX) \
			$(call GoPackage/Environment) \
			GITCOMMIT=$(PKG_SOURCE_VERSION) \
			DOCKER_GITCOMMIT=$(PKG_SOURCE_VERSION) \
			DOCKER_BUILDTAGS='$(BUILDTAGS)' \
			VERSION=$(PKG_VERSION) \
		\
		&& echo "Compiling CLI..." \
		&& cd $(PKG_BUILD_DIR)/cli \
		&& ./scripts/build/binary \
		\
		&& echo "Compiling Engine..." \
		&& cd $(PKG_BUILD_DIR)/engine \
		&& ./hack/make.sh binary \
	)

	# done here to include autogenerated files also
	rm $(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/cli
	rm $(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/docker
	$(call GoPackage/Build/Configure)
endef

define Package/docker-ce/install
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/cli/build/docker $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/engine/bundles/binary-daemon/dockerd $(1)/usr/bin/

	$(INSTALL_DIR) $(1)/opt/docker/
	$(INSTALL_DIR) $(1)/usr/share/docker/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/engine/contrib/check-config.sh $(1)/usr/share/docker/

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/dockerd.init $(1)/etc/init.d/dockerd
endef

$(eval $(call BuildPackage,docker-ce))
