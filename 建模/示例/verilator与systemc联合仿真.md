
## Verilator安装

以下安装均在ubuntu下进行
```bash
sudo apt-get install verilator
```

## 编写Verilog模型及systemc测试程序


### 编写verilog模型
下面以一个counter计数器为例,编写一个verilog模型

创建counter.v文件，并以下代码
```verilog
module counter(
	input clk,
	input rst,
	output reg[7:0] cnt
);
	always@(posedge clk or posedge rst)
	begin
		if(rst)
			cnt <= 0;
		else
			cnt <= cnt + 1;

	end
endmodule
```

### 编写systemc测试程序

注意，调用counter模块的时候，均需加上前缀`V`，如下代码
```cpp
#include "Vcounter.h"

int sc_main(int argc, char** argv){

	//Verilated::commandArgs(argc, argv);
	
	sc_clock clk{ "clk" , 10 , SC_NS , 0.5 , 3 , SC_NS, true };
	sc_signal<bool> reset;
	sc_signal<unsigned int> counter_out;

  //例化counter并连接数据线
	Vcounter top("counter") ; 
	top.clk( clk );
	top.rst(reset);
	top.cnt( counter_out);

	//reset
	reset = 1;
	for( int i = 0; i<100;i++){
		sc_start(10,SC_NS);
	}
	reset = 0;

	//读取counter值并输出
	for( int i = 0; i<100;i++){
		sc_start(10,SC_NS);	
		cout << counter_out << endl;
	}

	return 0;
}
```

##编译并运行测试程序
```bash
verilator --Wall --sc --exe counter_tb.cpp counter.v
make -j -C obj_dir/ -f Vcounter.mk Vcounter
./a.out
```

运行结果如下图所示:
![](./Verilator仿真结果.png)



