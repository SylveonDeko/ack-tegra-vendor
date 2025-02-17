# SPDX-FileCopyrightText: Copyright (c) 2022-2023 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: GPL-2.0-only

ifneq ($(TEGRA_TOP),)
SOURCE_TOP := $(TEGRA_TOP)
else ifneq ($(TOP),)
SOURCE_TOP := $(TOP)
else ifneq ($(ROOT_DIR),)
SOURCE_TOP := $(ROOT_DIR)
else
$(error TEGRA_TOP or TOP is not defined)
endif

tegra-dtstree := $(SOURCE_TOP)/vendor/nvidia/tegra/dts

null :=
space :=$(null) $(null)

DTC_INCLUDE := $(srctree)/arch/arm64/boot/dts/nvidia
# SOC independent common include
DTC_INCLUDE += $(tegra-dtstree)/tegra/nv-public
DTC_INCLUDE += $(tegra-dtstree)/tegra/nv-public/include/kernel
DTC_INCLUDE += $(tegra-dtstree)/tegra/nv-public/include/nvidia-oot

ifeq ($(CONFIG_ARCH_TEGRA_234_SOC),y)
# SOC T23X specific common include
DTC_INCLUDE += $(tegra-dtstree)/t23x/nv-public/include/kernel
DTC_INCLUDE += $(tegra-dtstree)/t23x/nv-public/include/nvidia-oot
DTC_INCLUDE += $(tegra-dtstree)/t23x/nv-public/include/platforms
DTC_INCLUDE += $(tegra-dtstree)/t23x/nv-public
endif

dtb-y :=
dtbo-y :=
fdts :=
ifeq ($(CONFIG_ARCH_TEGRA_210_SOC),y)
fdts += $(tegra-dtstree)/t21x
endif
ifeq ($(CONFIG_ARCH_TEGRA_186_SOC),y)
fdts += $(tegra-dtstree)/t18x
endif
ifeq ($(CONFIG_ARCH_TEGRA_194_SOC),y)
fdts += $(tegra-dtstree)/t19x
endif
ifeq ($(CONFIG_ARCH_TEGRA_234_SOC),y)
fdts += $(tegra-dtstree)/t23x
endif

# Add internal SOCs to scan the DT makefiles
ifneq ($(internal_soc_list),)
fdts += $(addprefix $(tegra-dtstree)/,$(internal_soc_list))
endif

# Remove the DTs from protected soc list
ifneq ($(protected_soc_list),)
kdts := $(foreach dt_path, $(fdts), $(if $(filter $(protected_soc_list),$(patsubst -,$(space),$(subst /, $(space),$(dt_path)))),,$(dt_path)))
kdts := $(filter-out $(space)$(space),$(kdts))
else
kdts := $(fdts)
endif

dts_makefile=$(foreach d,$(wildcard $1*), $(call dts_makefile,$(d)/,$(2)) $(if $(findstring Makefile,$(d)),$(d)))
dts_mfiles = $(call dts_makefile, $(kdts), Makefile)

ifneq ($(dts_mfiles),)
dts-include :=
include $(dts_mfiles)
ifneq ($(dts-include),)
DTC_INCLUDE += $(addprefix $(tegra-dtstree)/,$(dts-include))
endif
endif

# Add path of main Makefile to each dtb/dtbo list
DTB_OBJS := $(addprefix $(obj)/,$(dtb-y))
DTBO_OBJS := $(addprefix $(obj)/,$(dtbo-y))

define _define_dtb_rule
$(obj)/$(1): $(tegra-dtstree)/$(patsubst %.dtb,%.dts,$(1)) FORCE
endef

$(foreach _dtb, $(dtb-y), $(eval $(call _define_dtb_rule,$(_dtb))))

$(DTB_OBJS):
	$(call if_changed_dep,dtc)

define _define_dtbo_rule
$(obj)/$(1): $(tegra-dtstree)/$(patsubst %.dtbo,%.dts,$(1)) FORCE
endef

$(foreach _dtbo, $(dtbo-y), $(eval $(call _define_dtbo_rule,$(_dtbo))))

$(DTBO_OBJS):
	$(call if_changed_dep,dtc)

DTBS_DTBOS := $(DTB_OBJS)$(DTBO_OBJS)

dtbs: $(DTB_OBJS) $(DTBO_OBJS) FORCE
	if [ ! -d $(obj)/dtbs/ ] ; then \
		mkdir -p $(obj)/dtbs/ ; \
	fi
	if [ ! -z "$(DTBS_DTBOS)" ] ; then \
		cp -u $(DTB_OBJS) $(DTBO_OBJS) $(obj)/dtbs/ ; \
	fi
	if [ -d $(obj)/hardware/ ] ; then \
		rm -rf $(obj)/hardware/ ; \
	fi

dtb-y += $(dtbo-y)

clean-files := *.dtb *.dtbo *.tmp
