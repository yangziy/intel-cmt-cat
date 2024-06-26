###############################################################################
# Makefile script for CAT PQoS sample applications
#
# @par
# BSD LICENSE
#
# Copyright(c) 2018-2023 Intel Corporation. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
#   * Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#   * Redistributions in binary form must reproduce the above copyright
#     notice, this list of conditions and the following disclaimer in
#     the documentation and/or other materials provided with the
#     distribution.
#   * Neither the name of Intel Corporation nor the names of its
#     contributors may be used to endorse or promote products derived
#     from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
###############################################################################

include ../../../pre-build.mk

LIBDIR ?= ../../../lib
CFLAGS =-I$(LIBDIR) \
	-W -Wall -Wextra -Wstrict-prototypes -Wmissing-prototypes \
	-Wmissing-declarations -Wold-style-definition -Wpointer-arith \
	-Wcast-qual -Wundef -Wwrite-strings  \
	-Wformat -Wformat-security -fstack-protector-strong -fPIE -D_FORTIFY_SOURCE=2 \
	-Wunreachable-code -Wsign-compare -Wno-endif-labels \
	-g -O2
ifneq ($(EXTRA_CFLAGS),)
CFLAGS += $(EXTRA_CFLAGS)
endif
LDFLAGS=-L$(LIBDIR) -pie -z noexecstack -z relro -z now
LDLIBS=-lpqos -lpthread

# ICC and GCC options
ifeq ($(CC),icc)
else
CFLAGS += -Wcast-align \
    -Wnested-externs \
    -Wmissing-noreturn
endif

IS_GCC = $(shell $(CC) -v 2>&1 | grep -c "^gcc version ")
# GCC-only options
ifeq ($(IS_GCC),1)
CFLAGS += -fno-strict-overflow \
    -fno-delete-null-pointer-checks \
    -fwrapv \
    -fstack-clash-protection \
    -fcf-protection=full
endif

# Build targets and dependencies
ALLOCAPP_L2CAT = allocation_app_l2cat
ALLOCAPP_L3CAT = allocation_app_l3cat
ALLOCAPP_MBA = allocation_app_mba
ASSOCAPP = association_app
RESETAPP = reset_app

all: $(ALLOCAPP_L2CAT) $(ALLOCAPP_L3CAT) $(ALLOCAPP_MBA) $(ASSOCAPP) $(RESETAPP)

$(ALLOCAPP_L2CAT): $(ALLOCAPP_L2CAT).o
$(ALLOCAPP_L3CAT): $(ALLOCAPP_L3CAT).o
$(ALLOCAPP_MBA): $(ALLOCAPP_MBA).o
$(ASSOCAPP): $(ASSOCAPP).o
$(RESETAPP): $(RESETAPP).o

.PHONY: clean
clean:
	-rm -f $(ALLOCAPP_L2CAT) $(ALLOCAPP_L3CAT) $(ALLOCAPP_MBA) $(ASSOCAPP) $(RESETAPP) *.o

CHECKPATCH?=checkpatch.pl
.PHONY: checkpatch
checkpatch:
	$(CHECKPATCH) --no-tree --no-signoff --emacs \
	--ignore CODE_INDENT,INITIALISED_STATIC,LEADING_SPACE,SPLIT_STRING,UNSPECIFIED_INT \
	--ignore SPDX_LICENSE_TAG,CONST_STRUCT \
	-f allocation_app_l2cat.c allocation_app_l3cat.c -f allocation_app_mba.c \
        -f association_app.c -f reset_app.c

CLANGFORMAT?=clang-format
.PHONY: clang-format
clang-format:
	@for file in $(wildcard *.[ch]); do \
		echo "Checking style $$file"; \
		$(CLANGFORMAT) -style=file "$$file" | diff "$$file" - | tee /dev/stderr | [ $$(wc -c) -eq 0 ] || \
		{ echo "ERROR: $$file has style problems"; exit 1; } \
	done

CODESPELL?=codespell
.PHONY: codespell
codespell:
	$(CODESPELL) . -q 2 -L clos

.PHONY: style
style:
	$(MAKE) checkpatch
	$(MAKE) clang-format
	$(MAKE) codespell

CPPCHECK?=cppcheck
.PHONY: cppcheck
cppcheck:
	$(CPPCHECK) --enable=warning,portability,performance,unusedFunction,missingInclude \
	--std=c99 -I$(LIBDIR) --template=gcc \
	--suppress=missingIncludeSystem \
	allocation_app_l2cat.c allocation_app_l3cat.c allocation_app_mba.c association_app.c \
        reset_app.c
