PROJ_NAME=bootloader
STANDALONE = ..


SRCS = 	${shell find src -type f -name \*.c} \
		${shell find src -type f -name \*.cpp}  \
		${shell find src -type f -name \*.S}

LDSCRIPT ?= ${BSP_PATH}/linker/bootloader.ld

include ${STANDALONE}/common/bsp.mk
include ${STANDALONE}/common/riscv64-unknown-elf.mk
include ${STANDALONE}/common/standalone.mk

