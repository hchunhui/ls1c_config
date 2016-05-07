HOST=mipsel-linux
SOURCE=$(PWD)/pkg

DROPBEAR=dropbear-2016.73
IPTABLES=iptables-1.4.21
PPP=ppp-2.4.7
RP_PPPOE=rp-pppoe-3.11
OPENSSL=openssl-1.0.2h
OPENSSH=openssh-7.2p2
LIBPCAP=libpcap-1.7.4
SHADOWSOCKS_LIBEV=shadowsocks-libev-git
OPENVPN=openvpn-2.3.10
LZO=lzo-2.09

all: dropbear iptables rp_pppoe openssh shadowsocks-libev openvpn

out:
	mkdir -p out

dropbear: out
	tar xvf $(SOURCE)/$(DROPBEAR).tar.bz2
	cd $(DROPBEAR) && \
	./configure --host=$(HOST) --prefix=/usr && \
	make PROGRAMS="dropbear dbclient dropbearkey dropbearconvert scp" SCPPROGRESS=1 DESTDIR=$$PWD/install install
	tar Cczvf $(DROPBEAR)/install out/$@.tgz usr
	touch $@

iptables:
	tar xvf $(SOURCE)/$(IPTABLES).tar.bz2
	cd $(IPTABLES) && \
	./configure --host=$(HOST) --prefix=/usr && \
	make && make DESTDIR=$$PWD/install install
	tar Cczvf $(IPTABLES)/install out/$@.tgz usr --exclude="pkgconfig" --exclude "include"
	touch $@

ppp:
	tar xvf $(SOURCE)/$(PPP).tar.gz
	cd $(PPP) && \
	./configure && \
	sed -i 's/^FILTER=y//g' pppd/Makefile && \
	make CC=$(HOST)-gcc && \
	$(HOST)-strip pppd/pppd
	touch $@

rp_pppoe: ppp
	tar xvf $(SOURCE)/$(RP_PPPOE).tar.gz
	cd $(RP_PPPOE)/src && \
	./configure --host=$(HOST) && \
	make && \
	make DESTDIR=$$PWD/install install
	cp $(PPP)/pppd/pppd $(RP_PPPOE)/src/install/usr/sbin/
	tar Cczvf $(RP_PPPOE)/src/install out/$@.tgz usr #etc
	touch $@

OPENSSL_TARGET=linux-mips32
openssl:
	tar xvf $(SOURCE)/$(OPENSSL).tar.gz
	cd $(OPENSSL) && \
	patch -p1 < $(SOURCE)/openssl.patch && \
	CC=$(HOST)-gcc ./Configure $(OPENSSL_TARGET) shared --openssldir=$$PWD/install && \
	make depend && make && make install_sw
	tar Cczvf $(OPENSSL)/install out/$@.tgz lib --transform 's,^,usr/,' --exclude='*.a' --exclude='pkgconfig'
	touch $@

openssh: openssl
	tar xvf $(SOURCE)/$(OPENSSH).tar.gz
	cd $(OPENSSH) && \
	./configure --build=`./config.guess` --host=$(HOST) --with-ssl-dir=$$PWD/../$(OPENSSL)/install && \
	make && $(HOST)-strip ssh && $(HOST)-strip sshd
	cp $(OPENSSH)/ssh out/
	cp $(OPENSSH)/sshd out/
	touch $@

libpcap:
	tar xvf $(SOURCE)/$(LIBPCAP).tar.gz
	cd $(LIBPCAP) && \
	./configure --host=$(HOST) && \
	make
	touch $@

shadowsocks-libev: openssl
	cd $(SHADOWSOCKS_LIBEV) && \
	./configure --host=$(HOST) --with-openssl=$$PWD/../$(OPENSSL)/install && \
	make && $(HOST)-strip src/ss-local
	cp $(SHADOWSOCKS_LIBEV)/src/ss-local out/
	touch $@

lzo:
	tar xvf $(SOURCE)/$(LZO).tar.gz
	cd $(LZO) && \
	./configure --host=$(HOST) && \
	make
	touch $@

openvpn: openssl lzo
	tar xvf $(SOURCE)/$(OPENVPN).tar.gz
	cd $(OPENVPN) && \
	./configure --host=$(HOST) \
		--disable-plugin-auth-pam \
		LZO_CFLAGS="-I$$PWD/../$(LZO)/include" \
		LZO_LIBS="-L$$PWD/../$(LZO)/src/.libs -llzo2" \
		OPENSSL_SSL_CFLAGS="-I$$PWD/../$(OPENSSL)/install/include" \
		OPENSSL_SSL_LIBS="-L$$PWD/../$(OPENSSL)/install/lib -lssl" \
		OPENSSL_CRYPTO_CFLAGS="-I$$PWD/../$(OPENSSL)/install/include" \
		OPENSSL_CRYPTO_LIBS="-L$$PWD/../$(OPENSSL)/install/lib -lcrypto" && \
	make && $(HOST)-strip src/openvpn/openvpn
	cp $(OPENVPN)/src/openvpn/openvpn out/
	touch $@
