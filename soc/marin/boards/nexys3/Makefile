COREGEN_SOURCES = clk_wiz_v3_6.v fifo_generator_v9_2.v ram4k16bit.v 

SOURCES = $(wildcard ../../../../cores/MoxieLite/*.vhd) \
	  $(wildcard ../../../../cores/MoxieLite/*.v) \
	  $(wildcard ../../../../cores/nexys7seg/*.v) \
	  $(wildcard ../../../../cores/wishbone/*.v) \
	  $(wildcard ../../../../cores/uart/*.v) \
	  $(wildcard ../../../../cores/ram16bit/*.v) \
	  $(wildcard ../../../../cores/gdbtarget/*.v) \
	  $(wildcard ../../../../cores/statled/rtl/*.v) \
	  $(wildcard ../../rtl/*.v) \
	  $(COREGEN_SOURCES) 

#	  /opt/Xilinx/14.2/ISE_DS/ISE/verilog/src/iSE/unisim_comp.v

PROJECT = marin

all: $(PROJECT).bit

ram4k16bit.v: ram4k16bit.xco coregen.cgp
	coregen -b ram4k16bit.xco -p coregen.cgp

clk_wiz_v3_6.v: clk_wiz_v3_6.xco coregen.cgp
	coregen -b clk_wiz_v3_6.xco -p coregen.cgp

fifo_generator_v9_2.v: fifo_generator_v9_2.xco coregen.cgp
	coregen -b fifo_generator_v9_2.xco -p coregen.cgp

# CFLAGS = -Os -ffunction-sections -mno-crt0
CFLAGS = -Os  -mno-crt0

#BOOTROM_SOURCES = ../../../../firmware/boot.c ../../../../firmware/init.S
BOOTROM_SOURCES = ../../../../firmware/marin.S ../../../../firmware/mdata.c ../../../../firmware/handler.S

bootrom.vh: $(BOOTROM_SOURCES) ../../moxie-marin.ld
#	moxie-elf-gcc $(CFLAGS) -Wl,--gc-sections -T../../moxie-marin.ld -o bootrom.x $(BOOTROM_SOURCES)
	moxie-elf-gcc $(CFLAGS) -c -o marin.o ../../../../firmware/marin.S
	moxie-elf-gcc $(CFLAGS) -c -o mdata.o ../../../../firmware/mdata.c
	moxie-elf-gcc $(CFLAGS) marin.o mdata.o -o bootrom.x -T../../moxie-marin.ld -lnosys
#	moxie-elf-ld marin.o mdata.o -o bootrom.x -T../../moxie-marin.ld -Map=mapfile
	moxie-elf-objcopy -O verilog bootrom.x bootrom.vh

$(PROJECT).prj: bootrom.vh ../../../../cores/gdbtarget/messages.vh $(SOURCES)
	rm -f $(PROJECT).prj
	for i in `echo $^`; do \
	  case $$i in \
	    *.vhd) \
		echo "vhdl work $$i" >> $@; \
		;; \
	    *.v) \
		echo "verilog work $$i" >> $@; \
		;; \
	  esac; \
	done;

$(PROJECT).bit: $(PROJECT).ncd $(PROJECT).ut
	bitgen -f $(PROJECT).ut $(PROJECT).ncd

$(PROJECT).ngd: $(PROJECT).ngc
	ngdbuild -dd . -nt timestamp -uc Nexys3_Master.ucf -p xc6slx16-csg324-3 $(PROJECT).ngc $(PROJECT).ngd

$(PROJECT).ngc $(PROJECT).syr: $(PROJECT).prj $(PROJECT).xst
	xst -ifn $(PROJECT).xst -ofn $(PROJECT).syr

$(PROJECT)_map.ncd $(PROJECT).pcf: $(PROJECT).ngd
	map -w -o $(PROJECT)_map.ncd $(PROJECT).ngd $(PROJECT).pcf

$(PROJECT).ncd: $(PROJECT)_map.ncd $(PROJECT).pcf
	par -w $(PROJECT)_map.ncd $(PROJECT).ncd $(PROJECT).pcf

clean: 
	-rm -rf *.prj *.ngd *.srp *~ *.lso *.xrpt xst _xmsgs
	-rm -rf *.map *.par *.syr *.log *.ncd *.twr *xprt *.lso
	-rm -rf *.bld xlnx_auto_0_xdb *.mrp *.ngm *.ngc *.xml 
	-rm -rf *.lst *.pcf *.bgn *.bit *.xwbt *.drc *.csv
	-rm -rf *.txt *.ptwx *.unroutes *.xpi *.html *.pad
	-rm -rf _impact.cmd *.ipf auto_project* *.xsl *.svf
	-rm -rf tmp coregen.cgc bootrom.vh bootrom.x
	-rm -rf $(COREGEN_SOURCES) $(COREGEN_SOURCES:.v=.ucf)
	-rm -rf $(COREGEN_SOURCES:.v=.asy)
	-rm -rf $(COREGEN_SOURCES:.v=.gise)
	-rm -rf $(COREGEN_SOURCES:.v=.veo)
	-rm -rf $(COREGEN_SOURCES:.v=.xdc)
	-rm -rf $(COREGEN_SOURCES:.v=.xise)
	-rm -rf $(COREGEN_SOURCES:.v=_xmdf.tcl)
	-rm -rf $(COREGEN_SOURCES:.v=.cgc)
	-rm -rf $(COREGEN_SOURCES:.v=)
