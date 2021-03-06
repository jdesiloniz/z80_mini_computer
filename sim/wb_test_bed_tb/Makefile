.PHONY: all
.DELETE_ON_ERROR:
TOPMOD  := wb_test_bed
FIFOMOD := wb_fifo
UARXMOD := wb_uart_rx
UATXMOD := wb_uart_tx
MEMAMOD := wb_mem_adapter
CLDVMOD := clk_divider
SHFTMOD := shifter
RESTMOD := reset_controller
FIFOFIL := $(FIFOMOD).v
UARXFIL := $(UARXMOD).v
UATXFIL := $(UATXMOD).v
MEMAFIL := $(MEMAMOD).v
CLDVFIL := $(CLDVMOD).v
SHFVFIL := $(SHFTMOD).v
RESTFIL := $(RESTMOD).v
VLOGFIL := $(TOPMOD).v
VLOGDIR := ../../rtl
VCDFILE := $(TOPMOD).vcd
SIMPROG := $(TOPMOD)_tb
SIMFILE := $(SIMPROG).cpp
FIFOSIM := $(FIFOMOD).cpp
SIMPLUG := ../signals/signals.cpp
SIMINC := ../include
VDIRFB  := ./obj_dir
all: $(VCDFILE)

GCC := g++
CFLAGS = -g -Wall -I$(VINC) -I $(VDIRFB) -I $(SIMINC)
#
# Modern versions of Verilator and C++ may require an -faligned-new flag
# CFLAGS = -g -Wall -faligned-new -I$(VINC) -I $(VDIRFB)

VERILATOR=verilator
VFLAGS := -O3 -MMD --trace -Wall --top-module $(TOPMOD)

## Find the directory containing the Verilog sources.  This is given from
## calling: "verilator -V" and finding the VERILATOR_ROOT output line from
## within it.  From this VERILATOR_ROOT value, we can find all the components
## we need here--in particular, the verilator include directory
VERILATOR_ROOT ?= $(shell bash -c '$(VERILATOR) -V|grep VERILATOR_ROOT | head -1 | sed -e "s/^.*=\s*//"')
##
## The directory containing the verilator includes
VINC := $(VERILATOR_ROOT)/include

$(VDIRFB)/V$(TOPMOD).cpp: $(VLOGDIR)/$(VLOGFIL)
	$(VERILATOR) $(VFLAGS) -cc $(VLOGDIR)/$(CLDVFIL) $(VLOGDIR)/$(SHFVFIL) $(VLOGDIR)/$(FIFOFIL) $(VLOGDIR)/$(VLOGFIL) $(VLOGDIR)/$(UARXFIL) $(VLOGDIR)/$(UATXFIL) $(VLOGDIR)/$(MEMAFIL) $(VLOGDIR)/$(RESTFIL)

$(VDIRFB)/V$(TOPMOD)__ALL.a: $(VDIRFB)/V$(TOPMOD).cpp
	make --no-print-directory -C $(VDIRFB) -f V$(TOPMOD).mk

$(SIMPROG): $(SIMFILE) $(SIMPLUG) $(VDIRFB)/V$(TOPMOD)__ALL.a
	$(GCC) $(CFLAGS) $(VINC)/verilated.cpp				\
		$(VINC)/verilated_vcd_c.cpp $(SIMFILE) $(SIMPLUG)	\
		$(VDIRFB)/V$(TOPMOD)__ALL.a -o $(SIMPROG) 

test: $(VCDFILE)

$(VCDFILE): $(SIMPROG)
	./$(SIMPROG)

## 
.PHONY: clean
clean:
	rm -rf $(VDIRFB)/ $(SIMPROG) $(VCDFILE)

##
## Find all of the Verilog dependencies and submodules
##
DEPS := $(wildcard $(VDIRFB)/*.d)

## Include any of these submodules in the Makefile
## ... but only if we are not building the "clean" target
## which would (oops) try to build those dependencies again
##
ifneq ($(MAKECMDGOALS),clean)
ifneq ($(DEPS),)
include $(DEPS)
endif
endif