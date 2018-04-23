IMPORT_PATH:=github.com/bluemir/panok
BIN_NAME:=$(notdir $(IMPORT_PATH))

default: $(BIN_NAME)

GIT_COMMIT_ID:=$(shell git rev-parse --short HEAD)
VERSION:=$(GIT_COMMIT_ID)-$(shell date +"%Y%m%d.%H%M%S")

# if gopath not set, make inside current dir
ifeq ($(GOPATH),)
	GOPATH=$(PWD)/.GOPATH
endif

PROTO_SOURCE = $(shell find . -name ".GOPATH" -prune -o -type f -name '*.proto' -print)
GO_SOURCES = $(shell find . -name ".GOPATH" -prune -o -type f -name '*.go' -print)
JS_SOURCES = $(shell find app/js -type f -name '*.js' -print)
HTML_SOURCES = $(shell find app/html -type f -name '*.html' -print)
CSS_SOURCES = $(shell find app/less -type f -name '*.less' -print)
WEB_LIBS = $(shell find app/lib -type f -type f -print)

DISTS += $(HTML_SOURCES:app/html/%=dist/html/%)
DISTS += $(JS_SOURCES:app/js/%=dist/js/%)
DISTS += dist/css/common.css
DISTS += $(WEB_LIBS:app/lib/%=dist/lib/%)

# Automatic runner
DIRS = $(shell find . \
	   -name ".git" -prune -o \
	   -name ".GOPATH" -prune -o \
	   -name "vendor" -prune -o \
	   -type d -print)

.sources:
	@echo $(DIRS) makefile \
		$(GO_SOURCES) \
		$(JS_SOURCES) \
		$(HTML_SOURCES) \
		$(CSS_SOURCES) \
		$(WEB_LIBS)| tr " " "\n"
run: $(BIN_NAME)
	go test ./...
	./$(BIN_NAME)
auto-run:
	while true; do \
		make .sources | entr -rd make run ; \
		echo "hit ^C again to quit" && sleep 1 ; \
	done
reset:
	ps -e | grep make | grep -v grep | awk '{print $$1}' | xargs kill


## Binary build (trick)
$(BIN_NAME): build///$(BIN_NAME)
	cp $< $@

## protobuf(for grpc)
%.pb.go: %.proto
	protoc -I . $< --go_out=plugins=grpc:.

## Web dist
dist/css/%.css: $(CSS_SOURCES)
	lessc app/less/main.less $@
dist/%: app/%
	@mkdir -p $(basename $@)
	cp $< $@

tools:
	npm install -g less
	go get github.com/GeertJohan/go.rice/rice
clean:
	rm -rf build/ dist/ vendor/ \
		$(BIN_NAME) $(BIN_NAME).bin $(BIN_NAME).tmp \
		$(PROTO_SOURCE:%.proto=%.pb.go)
	go clean

$(GOPATH)/src/$(IMPORT_PATH):
	@echo "make symbolic link on $(GOPATH)/src/$(IMPORT_PATH)..."
	@mkdir -p $(dir $(GOPATH)/src/$(IMPORT_PATH))
	ln -s $(PWD) $(GOPATH)/src/$(IMPORT_PATH)

## Multi platform build

deploy: build/linux/amd64/$(BIN_NAME)
deploy: build/linux/arm/$(BIN_NAME)
deploy: build/windows/amd64/$(BIN_NAME)
#deploy: build/osx/arm/$(BIN_NAME)
deploy:
	# TODO scp or upload binary
	# TODO call hook to deploy(ex. docker command)

## Binary build
build/%/$(BIN_NAME): build/%/$(BIN_NAME).bin $(DISTS)
	@echo --------------------------EMBED $$GOOS $$GOARCH-----------------------------
	cp $@.bin $@.tmp
	#rice append -v --exec $(BIN_NAME).tmp \
	#	-i $(IMPORT_PATH)/http-server
	mv $@.tmp $@
	@echo Embed resources DONE

build/%/$(BIN_NAME).bin: export GOOS=$(subst /,,$(dir $*))
build/%/$(BIN_NAME).bin: export GOARCH=$(notdir $*)
build/%/$(BIN_NAME).bin: $(GOPATH)/src/$(IMPORT_PATH) $(GO_SOURCES) $(PROTO_SOURCE:%.proto=%.pb.go)
	@echo --------------------------BUILD $$GOOS $$GOARCH-----------------------------
	mkdir -p $(@D)
	go get -v -d $(IMPORT_PATH)            # can replace with glide
	go build -i -v \
		-ldflags "-X main.VERSION=$(VERSION)" \
		-ldflags "-extldflags -static" \
		-o $@ .
	@echo Build DONE

# uncomment if feel like not working
#.PRECIOUS: build/%/$(BIN_NAME).bin
.PHONY: .sources run auto-run reset tools clean
