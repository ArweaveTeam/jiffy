# Based on c_src.mk from erlang.mk by Loic Hoguin <essen@ninenines.eu>

CURDIR := $(shell pwd)
BASEDIR := $(abspath $(CURDIR)/..)

PROJECT = jiffy

ERTS_INCLUDE_DIR ?= $(shell erl -noshell -eval 'io:format("~ts/erts-~ts/include/", [code:root_dir(), erlang:system_info(version)]).' -s init stop)
ERL_INTERFACE_INCLUDE_DIR ?= $(shell erl -noshell -eval 'io:format("~ts", [code:lib_dir(erl_interface, include)]).' -s init stop)
ERL_INTERFACE_LIB_DIR ?= $(shell erl -noshell -eval 'io:format("~ts", [code:lib_dir(erl_interface, lib)]).' -s init stop)

C_SRC_DIR = $(CURDIR)
C_SRC_OUTPUT ?= $(CURDIR)/../priv/$(PROJECT).so

# System type and C compiler/flags.

UNAME_SYS := $(shell uname -s)
ifeq ($(UNAME_SYS), Darwin)
	OSX_CPU_ARCH ?= x86_64
	# nix systems may not have sysctl where uname -m will return the correct arch
	SYSCTL_EXISTS := $(shell which sysctl 2>/dev/null)
	ifdef SYSCTL_EXISTS
		ifneq ($(shell sysctl -n machdep.cpu.brand_string | egrep "M(1|2)"),)
	  	OSX_CPU_ARCH = arm64
		endif
  else
		ifneq ($(shell uname -m | egrep "arm64"),)
	  	OSX_CPU_ARCH = arm64
	 	endif
  endif
	CC ?= cc
	CFLAGS ?= -O3 -std=c99 -arch $(OSX_CPU_ARCH) -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O3 -arch $(OSX_CPU_ARCH) -Wall
	LDFLAGS ?= -arch $(OSX_CPU_ARCH) -flat_namespace -undefined suppress
else ifeq ($(UNAME_SYS), FreeBSD)
	CC ?= cc
	CFLAGS ?= -O3 -std=c99 -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O3 -Wall
else ifeq ($(UNAME_SYS), Linux)
	CC ?= gcc
	CFLAGS ?= -fPIC -O3 -std=c99 -Wall -Wmissing-prototypes
	CXXFLAGS ?= -fPIC -O3 -Wall
	LDFLAGS ?= -shared
endif

CFLAGS += -I $(ERTS_INCLUDE_DIR) -I $(ERL_INTERFACE_INCLUDE_DIR)
CXXFLAGS += -I $(ERTS_INCLUDE_DIR) -I $(ERL_INTERFACE_INCLUDE_DIR)

LDLIBS += -L $(ERL_INTERFACE_LIB_DIR) -ldl -lpthread -lei -lstdc++

# Verbosity.

c_verbose_0 = @echo " C     " $(?F);
c_verbose = $(c_verbose_$(V))

cpp_verbose_0 = @echo " CPP   " $(?F);
cpp_verbose = $(cpp_verbose_$(V))

link_verbose_0 = @echo " LD    " $(@F);
link_verbose = $(link_verbose_$(V))

SOURCES := $(shell find $(C_SRC_DIR) -type f \( -name "*.c" -o -name "*.C" -o -name "*.cc" -o -name "*.cpp" \))
OBJECTS = $(addsuffix .o, $(basename $(SOURCES)))

COMPILE_C = $(c_verbose) $(CC) $(CFLAGS) $(CPPFLAGS) -c
COMPILE_CPP = $(cpp_verbose) $(CXX) $(CXXFLAGS) $(CPPFLAGS) -c

$(C_SRC_OUTPUT): $(OBJECTS)
	@mkdir -p $(BASEDIR)/priv/
	$(link_verbose) $(CC) $(OBJECTS) $(LDFLAGS) $(LDLIBS) -o $(C_SRC_OUTPUT)

%.o: %.c
	$(COMPILE_C) $(OUTPUT_OPTION) $<

%.o: %.cc
	$(COMPILE_CPP) $(OUTPUT_OPTION) $<

%.o: %.C
	$(COMPILE_CPP) $(OUTPUT_OPTION) $<

%.o: %.cpp
	$(COMPILE_CPP) $(OUTPUT_OPTION) $<

clean:
	@rm -f $(C_SRC_OUTPUT) $(OBJECTS)
