processors := arm hg51b upd96050 gsu r65816 spc700 lr35902
include processor/Makefile

include sfc/Makefile
include gb/Makefile
output := libretro

core_installdir := $(prefix)/lib

ifeq ($(platform),linux)
  flags += -fPIC
else ifeq ($(platform),osx)
  flags += -fPIC
else ifneq (,$(findstring ios,$(platform)))

ifeq ($(IOSSDK),)
   IOSSDK := $(shell xcodebuild -version -sdk iphoneos Path)
endif
ifeq ($(platform),ios-arm64)
	flags += -arch arm64 -fPIC -isysroot $(IOSSDK) -marm -DHAVE_POSIX_MEMALIGN=1 -w -DIOS
else
  flags += -arch armv7 -fPIC -isysroot $(IOSSDK) -marm -DHAVE_POSIX_MEMALIGN=1 -w -DIOS
endif
ifeq ($(platform),$(filter $(platform),ios9 ios-arm64))
	flags += -miphoneos-version-min=8.0
else
	flags += -miphoneos-version-min=5.0
endif
endif

flags += -D__LIBRETRO__

GIT_VERSION ?= " $(shell git rev-parse --short HEAD || echo unknown)"
ifneq ($(GIT_VERSION)," unknown")
	flags += -DGIT_VERSION=\"$(GIT_VERSION)\"
endif

#rules
objects := $(patsubst %,obj/%.o,$(objects))
sfc_objects += libretro
sfc_objects := $(patsubst %,obj/%-$(profile).o,$(sfc_objects))
objects += $(sfc_objects)

obj/libretro-$(profile).o: $(ui)/libretro.cpp $(ui)/*

#targets
build: $(objects)
ifeq ($(platform),linux)
	$(compiler) -o out/bsnes_$(profile)_libretro.so -shared $(objects) -ldl -Wl,--no-undefined -Wl,--version-script=$(ui)/link.T
else ifneq (,$(findstring ios,$(platform)))
  ifeq ($(platform),ios-arm64)
		$(compiler) -o out/bsnes_$(profile)_libretro_ios.dylib -dynamiclib $(objects) -isysroot $(IOSSDK) -arch arm64
  else
		$(compiler) -o out/bsnes_$(profile)_libretro_ios.dylib -dynamiclib $(objects) -isysroot $(IOSSDK) -arch armv7
  endif
else ifeq ($(platform),macosx)
ifdef ($(NOUNIVERSAL))
	$(compiler) -o out/bsnes_$(profile)_libretro.dylib -dynamiclib $(objects)
else
	$(compiler) $(ARCHFLAGS) -o out/bsnes_$(profile)_libretro.dylib -dynamiclib $(objects)
endif
else ifeq ($(platform),windows)
	$(compiler) -o out/bsnes_$(profile)_libretro.dll -shared $(objects) -Wl,--no-undefined -Wl,--version-script=$(ui)/link.T -static-libgcc -static-libstdc++ -lws2_32
else
	$(error invalid platform: $(platform). valid platforms are: windows linux macosx ios bsd)
endif

install:
ifeq ($(platform),linux)
	install -D -m 755 out/bsnes_$(profile)_libretro.so $(DESTDIR)$(core_installdir)/bsnes_$(profile)_libretro.so
else ifeq ($(platform),macosx)
	cp out/bsnes_$(profile)_libretro.dylib $(DESTDIR)$(core_installdir)/bsnes_$(profile)_libretro.dylib
else ifneq (,$(findstring ios,$(platform)))
	cp out/bsnes_$(profile)_libretro_ios.dylib $(DESTDIR)$(core_installdir)/bsnes_$(profile)_libretro_ios.dylib
endif

uninstall:
ifeq ($(platform),linux)
	rm $(DESTDIR)$(core_installdir)/bsnes_$(profile)_libretro.so
else ifeq ($(platform),macosx)
	rm $(DESTDIR)$(core_installdir)/bsnes_$(profile)_libretro.dylib
else ifneq (,$(findstring ios,$(platform)))
	rm $(DESTDIR)$(core_installdir)/bsnes_$(profile)_libretro_ios.dylib
endif

