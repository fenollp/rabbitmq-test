OPENSSL=openssl

ifndef DIR
DIR := .
endif

ifdef PASSWORD
P12PASS := true
else
P12PASS := @echo No PASSWORD defined. && false
endif

.PRECIOUS: %/testca
.PHONY: %/clean target all p12pass

all: client server

client server: p12pass
	@echo $(DIR)
	$(MAKE) target DIR=$(DIR) TARGET=$@ EXTENSIONS=$@_ca_extensions

p12pass:
	$(P12PASS)

target: $(DIR)/testca
	mkdir -p $(DIR)/$(TARGET)
	{ ( cd $(DIR)/$(TARGET) && \
	    openssl genrsa -out key.pem 2048 &&\
	    openssl req -new -key key.pem -out req.pem -outform PEM\
		-subj /CN=$$(hostname)/O=$(TARGET)/L=$$$$/ -nodes &&\
	    cd ../testca && \
	    openssl ca -config openssl.cnf -in ../$(TARGET)/req.pem -out \
	      ../$(TARGET)/cert.pem -notext -batch -extensions \
	      $(EXTENSIONS) && \
	    cd ../$(TARGET) && \
	    openssl pkcs12 -export -out keycert.p12 -in cert.pem -inkey key.pem \
	      -passout pass:$(PASSWORD) ) || (rm -rf $(DIR)/$(TARGET) && false); }

$(DIR)/testca:
	mkdir -p $(DIR)/testca
	cp openssl.cnf $(DIR)/testca/openssl.cnf
	{ ( cd $(DIR)/testca && \
	    mkdir -p certs private && \
	    chmod 700 private && \
	    echo 01 > serial && \
	    touch index.txt && \
	    openssl req -x509 -config openssl.cnf -newkey rsa:2048 -days 365 \
	      -out cacert.pem -outform PEM -subj /CN=MyTestCA/L=$$$$/ -nodes && \
	    openssl x509 -in cacert.pem -out cacert.cer -outform DER ) \
	  || (rm -rf $@ && false); }

clean:
	rm -rf $(DIR)/testca
	rm -rf $(DIR)/server
	rm -rf $(DIR)/client
