CONTIKI_PROJECT = cetic-bridge

# Default mode selection

CETIC_6LBR_SMARTBRIDGE=0
CETIC_6LBR_TRANSPARENTBRIDGE=0
CETIC_6LBR_ROUTER=0

WITH_WEBSERVER=1
WITH_WEBSERVER_LITE=0

ECONOTAG_NODE_INFO=0
ECONOTAG_UDPSERVER=0
ECONOTAG_WEBSERVER_LITE=1

# End of user configurable section

all: $(CONTIKI_PROJECT)

UIP_CONF_IPV6=1

CONTIKI = ../..

PROJECTDIRS += 6lbr platform/econotag platform/native apps/webserver apps/udp-server apps/node-info

CFLAGS += -DPROJECT_CONF_H=\"project-conf.h\"

CFLAGS +=-DCETIC_6LBR_SMARTBRIDGE=$(CETIC_6LBR_SMARTBRIDGE) -DCETIC_6LBR_TRANSPARENTBRIDGE=$(CETIC_6LBR_TRANSPARENTBRIDGE) -DCETIC_6LBR_ROUTER=$(CETIC_6LBR_ROUTER)

ifneq ($(WITH_WEBSERVER_LITE),0)
WITH_WEBSERVER=0
endif

ifneq ($(WITH_WEBSERVER),0)
CFLAGS += -DWEBSERVER=1
PROJECT_SOURCEFILES += webserver.c httpd-simple.c 
endif

ifneq ($(WITH_WEBSERVER_LITE),0)
CFLAGS += -DWEBSERVER=1
PROJECT_SOURCEFILES += webserver-lite.c httpd-simple.c 
endif

WITH_UDPSERVER=1
ifneq ($(WITH_UDPSERVER),0)
CFLAGS += -DUDPSERVER=1
PROJECT_SOURCEFILES += udp-server.c 
endif

WITH_NODE_INFO=1
ifneq ($(WITH_NODE_INFO),0)
CFLAGS += -DCETIC_NODE_INFO=1
PROJECT_SOURCEFILES += node-info.c 
endif

PROJECT_SOURCEFILES += rio.c packet-filter.c sicslow-ethernet.c nvm-config.c node-info.c
ifeq ($(TARGET),native)
APPS = slip-cmd
PROJECT_SOURCEFILES += native-init.c native-nvm.c raw-tap-eth-drv.c raw-tap-dev.c native-rdc.c slip-cmds.c slip-config.c slip-dev.c
else
ifeq ($(TARGET),econotag)
PROJECT_SOURCEFILES += econotag-init.c econotag-nvm.c enc28j60.c enc28j60-drv.c
endif
endif

NATIVE_CONF=WITH_WEBSERVER=$(WITH_WEBSERVER) WITH_UDPSERVER=$(WITH_UDPSERVER) WITH_NODE_INFO=$(WITH_NODE_INFO)
ECONOTAG_CONF=WITH_WEBSERVER=0 WITH_WEBSERVER_LITE=$(ECONOTAG_WEBSERVER_LITE) WITH_UDPSERVER=$(ECONOTAG_UDPSERVER) WITH_NODE_INFO=$(ECONOTAG_NODE_INFO)
all_native:
	mkdir -p bin
	$(MAKE) TARGET=native clean
	$(MAKE) TARGET=native CETIC_6LBR_SMARTBRIDGE=1 $(NATIVE_CONF)
	mv cetic-bridge.native bin/cetic_6lbr_smart_bridge 
	$(MAKE) TARGET=native clean
	$(MAKE) TARGET=native CETIC_6LBR_TRANSPARENTBRIDGE=1 $(NATIVE_CONF)
	mv cetic-bridge.native bin/cetic_6lbr_transparent_bridge 
	$(MAKE) TARGET=native clean
	$(MAKE) TARGET=native CETIC_6LBR_ROUTER=1 $(NATIVE_CONF)
	mv cetic-bridge.native bin/cetic_6lbr_router 

all_econotag:
	mkdir -p bin_econotag
	$(MAKE) TARGET=econotag clean
	$(MAKE) TARGET=econotag CETIC_6LBR_SMARTBRIDGE=1 $(ECONOTAG_CONF)
	mv cetic-bridge_econotag.bin bin_econotag/cetic_6lbr_smart_bridge 
	$(MAKE) TARGET=econotag clean
	$(MAKE) TARGET=econotag CETIC_6LBR_TRANSPARENTBRIDGE=1 $(ECONOTAG_CONF)
	mv cetic-bridge_econotag.bin bin_econotag/cetic_6lbr_transparent_bridge 
	$(MAKE) TARGET=econotag clean
	$(MAKE) TARGET=econotag CETIC_6LBR_ROUTER=1 $(ECONOTAG_CONF)
	mv cetic-bridge_econotag.bin bin_econotag/cetic_6lbr_router 

VERSION=$(word 2,$(shell awk '/Version:/' package/DEBIAN/control))
PKG=6lbr-$(VERSION)
IPK=6lbr-$(VERSION)-ipk

dist:
	rm -rf $(PKG)
	cp -R package $(PKG)
	mkdir -p $(PKG)/usr/lib/6lbr/bin
	cp bin/* $(PKG)/usr/lib/6lbr/bin

deb: dist
	dpkg --build $(PKG) .

ipk: dist
	rm -rf $(IPK)
	rm -f $(IPK).ipk
	mkdir $(IPK)
	cd $(PKG) && tar zcvf ../$(IPK)/data.tar.gz --exclude=DEBIAN --exclude='*~' *
	cd control-ipk && tar zcvfh ../$(IPK)/control.tar.gz --exclude='*~' *
	echo 2.0 > $(IPK)/debian-binary
	cd $(IPK) && ar rc ../$(IPK).ipk debian-binary data.tar.gz control.tar.gz

include $(CONTIKI)/Makefile.include