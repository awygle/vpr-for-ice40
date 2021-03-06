VPR ?= vpr
YOSYS ?= yosys

ifneq (,$(ARCH))
include Makefile.$(ARCH)
else
$(warning "You probably want to define $$ARCH.")
endif

ifeq (,$(ARCH_DIR))
$(error "Please define $$ARCH_DIR to point to your architecture directory.")
endif
ifeq (,$(DEVICE))
$(error "Please define $$DEVICE to a value in your architecture file.")
endif
ifeq (,$(YOSYS_SCRIPT))
$(error "Please define $$YOSYS_SCRIPT to the commands needed for synthesis.")
endif

ARCH_DIR_FULL = $(shell realpath $(ARCH_DIR)/$(ARCH_TYPE))

ARCH_FILE = $(ARCH_DIR_FULL)/merged.xml
$(ARCH_FILE):
	cd $(ARCH_DIR_FULL); make merged.xml

OUTPUT_DIR ?= build/$(ARCH)/$(DEVICE)

$(OUTPUT_DIR):
	mkdir -p $@

# Yosys command
$(OUTPUT_DIR)/%.blif: %.v | $(OUTPUT_DIR)
	$(YOSYS) -p "$(YOSYS_SCRIPT) opt_clean; write_blif $@" $<

# We just have a BLIF file
$(OUTPUT_DIR)/%.blif: %.blif | $(OUTPUT_DIR)
	cp $< $@

# VPR commands
VPR_CMD = $(VPR) $(ARCH_FILE) --device $(DEVICE)
VPR_FULL_CMD = cd $(OUTPUT_DIR); $(VPR_CMD)

$(OUTPUT_DIR)/%.rr_graph.xml: $(OUTPUT_DIR)/%.blif $(ARCH_FILE)
	$(VPR_FULL_CMD) $(notdir $<) --write_rr_graph $(notdir $@)

%.disp: $(OUTPUT_DIR)/%.blif $(ARCH_FILE)
	$(VPR_FULL_CMD) $(notdir $<) --disp on

%.echo: $(OUTPUT_DIR)/%.blif $(ARCH_FILE)
	$(VPR_FULL_CMD) $(notdir $<) --echo_file on

%.gdb: $(OUTPUT_DIR)/%.blif $(ARCH_FILE)
	cd $(OUTPUT_DIR); gdb --args $(VPR_CMD) $(notdir $<) --echo_file on --disp on


ALL_OUTPUT = $(patsubst %.v,$(OUTPUT_DIR)/%.rr_graph.xml,$(wildcard *.v)) $(patsubst %.blif,$(OUTPUT_DIR)/%.rr_graph.xml,$(wildcard *.blif))
all: $(ALL_OUTPUT)
	@echo $(ALL_OUTPUT)

help:
	@echo ""
	@echo "For each blif file in this directory you can:"
	@echo " make xx.disp 		- Display the vpr process."
	@echo " make xx.echo 		- Generate useful debugging echo files."
	@echo " make xx.gdb  		- Run vpr process under gdb."
	@echo " make xx.rr_graph.xml	- Generate rr_graph.xml file."
	@echo ""
	@echo "You can set the following"
	@echo " ARCH=<directory containing merged.xml architecture definition>"
	@echo " VPR=<path to vpr>"
	@echo " YOSYS=<path to yosys>"
	@echo ""

clean:
	rm -rf build

.PHONY: all clean help %.gdb %.echo %.disp $(ARCH_FILE)
