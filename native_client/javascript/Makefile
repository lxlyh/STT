NODE_BUILD_TOOL    ?= node-pre-gyp
NODE_ABI_TARGET    ?= 
NODE_BUILD_VERBOSE ?= --verbose
NPM_TOOL           ?= npm
PROJECT_NAME       ?= @mozilla-voice/stt
PROJECT_VERSION    ?= $(shell cat ../../training/mozilla_voice_stt_training/VERSION | tr -d '\n')
NPM_ROOT           ?= $(shell npm root)

NODE_MODULES_BIN   ?= $(NPM_ROOT)/.bin/
ifeq ($(findstring _NT,$(OS)),_NT)
# On Windows, we seem to need both in PATH for node-pre-gyp as well as tsc
# they do not get installed the same way.
NODE_MODULES_BIN   := $(shell cygpath -u $(NPM_ROOT)/.bin/):$(shell cygpath -u `dirname "$(NPM_ROOT)"`)
endif

include ../definitions.mk

ifeq ($(findstring _NT,$(OS)),_NT)
LIBS := '$(shell cygpath -w $(subst .lib,,$(LIBS)))'
endif

.PHONY: npm-dev

default: build

clean:
	rm -f deepspeech_wrap.cxx package.json package-lock.json
	rm -rf ./build/

clean-npm-pack:
	rm -fr ./$(PROJECT_NAME)-*.tgz

really-clean: clean clean-npm-pack
	rm -fr ./node_modules/
	rm -fr ./lib/

package.json: package.json.in
	sed \
		-e 's`$$(PROJECT_NAME)`$(PROJECT_NAME)`' \
		-e 's/$$(PROJECT_VERSION)/$(PROJECT_VERSION)/' \
		package.json.in > package.json && cat package.json

npm-dev: package.json
	$(NPM_TOOL) install --prefix=$(NPM_ROOT)/../ --ignore-scripts --force --verbose --production=false .

configure: deepspeech_wrap.cxx package.json npm-dev
	PATH="$(NODE_MODULES_BIN):${PATH}" $(NODE_BUILD_TOOL) configure $(NODE_BUILD_VERBOSE)

build: configure deepspeech_wrap.cxx
	PATH="$(NODE_MODULES_BIN):${PATH}" AS=$(AS) CC=$(CC) CXX=$(CXX) LD=$(LD) CFLAGS="$(CFLAGS)" CXXFLAGS="$(CXXFLAGS)" LDFLAGS="$(RPATH_NODEJS) $(LDFLAGS)" LIBS=$(LIBS) $(NODE_BUILD_TOOL) $(NODE_PLATFORM_TARGET) $(NODE_RUNTIME) $(NODE_ABI_TARGET) $(NODE_DIST_URL) --no-color rebuild $(NODE_BUILD_VERBOSE)

copy-deps: build
	$(call copy_missing_libs,lib/binding/*/*/*/deepspeech.node,lib/binding/*/*/)

node-wrapper: copy-deps build
	PATH="$(NODE_MODULES_BIN):${PATH}" $(NODE_BUILD_TOOL) $(NODE_PLATFORM_TARGET) $(NODE_RUNTIME) $(NODE_ABI_TARGET) $(NODE_DIST_URL) --no-color package $(NODE_BUILD_VERBOSE)

npm-pack: clean package.json index.ts npm-dev
	PATH="$(NODE_MODULES_BIN):${PATH}" tsc && $(NPM_TOOL) pack $(NODE_BUILD_VERBOSE)

deepspeech_wrap.cxx: deepspeech.i ds-swig
	$(DS_SWIG_ENV) swig -c++ -javascript -node deepspeech.i
