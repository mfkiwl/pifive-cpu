PYTHON_SOURCES=$(shell find ./rtl -name "*.py")
THIRD_PARTY_SOURCES=./third_party/verilog-i2c/rtl/i2c_master.v ./third_party/verilog-i2c/rtl/axis_fifo.v ./third_party/verilog-i2c/rtl/i2c_master_wbs_16.v ./third_party/spi-controller/spi_controller.v ./third_party/hyperram/hyper_xface.v
VERILOG_SOURCES=$(shell find ./rtl/verilog -name "*.v") $(shell find ./rtl/verilog -name "*.sv") $(THIRD_PARTY_SOURCES)

build/top.v: build/build_top.v build/cpu_top.v build/MemorySubsystem.v $(VERILOG_SOURCES)
	mkdir -p build
	sv2v build/build_top.v build/cpu_top.v build/MemorySubsystem.v $(VERILOG_SOURCES) --exclude=assert > build/top.v

build/top_verif.v: build/build_top_verif.v build/cpu_top_verif.v build/MemorySubsystem.v $(VERILOG_SOURCES)
	mkdir -p build
	sv2v build/build_top_verif.v build/cpu_top_verif.v build/MemorySubsystem.v $(VERILOG_SOURCES) --exclude=assert --define=VERIFICATION --define=VERILATOR > build/top_verif.v

build/build_top_verif.v: $(PYTHON_SOURCES)
	mkdir -p build
	python3 rtl/build.py verif

build/build_top.v: $(PYTHON_SOURCES)
	mkdir -p build
	python3 rtl/build.py main

.PHONY: build/cpu_top.v
build/cpu_top.v:
	mkdir -p build
	make -C ../cpu build/top.v
	cp ../cpu/build/top.v build/cpu_top.v

.PHONY: build/cpu_top_verif.v
build/cpu_top_verif.v:
	mkdir -p build
	make -C ../cpu build/top_verif.v
	cp ../cpu/build/top_verif.v build/cpu_top_verif.v

.PHONY: est-util
est-util: build/top.v
	yosys -p 'read_verilog -sv build/top.v; synth -top soc; stat'

.PHONY: build/MemorySubsystem.v
build/MemorySubsystem.v:
	mkdir -p build
	make -C memory-subsystem build/MemorySubsystem.v > build/scala-build.log
	cp memory-subsystem/build/MemorySubsystem.v build/MemorySubsystem.v

.PHONY: build/soc_generated.v
build/soc_generated.v: build/top.v
	cp build/top.v build/soc_generated.v
	sed -i 's^$$display^begin end //$$display^g' build/soc_generated.v
	echo "" >> build/soc_generated.v
	echo "\`default_nettype wire" >> build/soc_generated.v
