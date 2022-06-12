## **Edalize**

### **What's this?**

Edalize는 EDA 도구와 상호 작용하기 위한 Python 라이브러리입니다. 지원되는 도구에 대한 프로젝트 파일을 생성하고 배치 또는 GUI 모드(지원되는 경우)에서 실행할 수 있습니다.

수상 경력에 빛나는 [Edalize 소개 영상](https://www.youtube.com/watch?v=HuRtkpZqB34)

Icarus, Yosys, ModelSim, Vivado, Verilator, GHDL, Quartus 등과 같은 모든 EDA 도구는 입력 HDL 파일(Verilog 및 VHDL)과 일부 도구별 파일(제약 파일, 메모리 초기화 파일, IP 설명 파일 등)을 받습니다. 파일과 함께 몇 가지 Verilog가 `정의하고 일부 최상위 매개변수/제네릭 또는 일부 도구별 옵션이 설정됩니다. 구성이 완료되면 시뮬레이션 모델, 넷리스트 또는 FPGA 이미지가 구축되고 시뮬레이션의 경우 일부 추가 런타임 매개변수와 함께 모델도 실행됩니다.

문제는 이러한 모든 도구가 완전히 다른 방식으로 이 작업을 수행하며 일반적으로 한 시뮬레이터에서 다른 시뮬레이터로 구성을 가져올 수 있는 방법이 없다는 것입니다.

두려워하지 마십시오! Edalize가 이를 처리합니다. 일부 정보와 함께 어떤 파일이 있는지, 컴파일 및 런타임에 사용할 매개변수화(예: plusargs, 정의, 제네릭, 매개변수), VPI 라이브러리 소스(해당되는 경우) 및 이미 언급했듯이 필요한 프로젝트 파일을 만들고 빌드 및 실행을 제안합니다.

이렇게 하면 EDA 도구를 직접 인터페이스하는 지루한 작업을 처리하지 않아도 되며 원하는 방식으로 프로젝트를 설정할 수 있는 충분한 권한을 갖게 됩니다.

최소한 시뮬레이터와 관련하여 도구를 빠르게 전환할 수 있습니다. 이것은 도구별 버그를 제거하거나 선택한 무기로 작업할 수 있도록 하는 데 매우 유용합니다.

또한 도구의 GUI에서 열 수 있는 빠른 템플릿을 얻고 거기에서 계속 작업하는 데 사용할 수도 있습니다.

기존 Python 기반 HDL 프로젝트에 대한 라이브러리로 직접 통합하거나 독립 실행형(어쨌든 조만간)을 사용하여 다른 언어로 작성된 프로젝트에서 Edalize를 공급할 수 있습니다.

### **Install it**

Edalize는 Python 모듈입니다. [github.com/olofk/edalize](https://github.com/olofk/edalize) 에서 소스를 찾으십시오 . 다운로드가 완료되면 다음 Python 명령으로 설치할 수 있습니다.

```
$ cd edalize
$ python -m pip install -e .
```

보고 모듈은 데이터 분석을 위한 여러 종속성을 사용하기 때문에 선택 사항이 되었습니다. 다음과 같이 설치할 수 있습니다.

```
$ python -m pip install -e ".[reporting]"
```

### **How to use it?**

좋아, 잘 들린다. 이제 시작하려면 어떻게 해야 합니까? [edalize.rtfd.io](https://edalize.rtfd.io/) 에서 문서를 찾으십시오 .

라는 Verilog 소스 파일로 구성된 프로젝트가 있다고 가정합니다 `blinky.v`. 그런 다음 테스트벤치라고 하는 테스트벤치 `blinky_tb.v`와 라는 합성을 위한 제약 파일 도 `constraints.sdc`있습니다. 해당 파일은 깜박임 및 [orpsoc](https://github.com/fusesoc/blinky) [-cores](https://github.com/fusesoc/vlog_tb_utils/blob/master/vlog_tb_utils.v)`vlog_tb_utils.v` 에서 가져올 수 있습니다 .

시뮬레이션을 위해 두 개의 Verilog 파일을 사용하고 이라는 하위 디렉토리에 빌드 `build`한 다음 매개변수와 함께 실행하여 시뮬레이션된 클럭 주파수를 제어하려고 합니다.

Edalize는 Python 도구이므로 Python 스크립트 파일 내에서 또는 Python 콘솔에서 직접 실행할 수 있습니다.

먼저 Edalize 객체를 가져와야 합니다.

```
from edalize import *
```

이 자습서에는 os 모듈도 필요합니다.

```
import os
```

그런 다음 사용할 파일을 등록합니다.

```
work_root = 'build'

files = [
  {'name' : os.path.relpath('blinky.v', work_root),
   'file_type' : 'verilogSource'},
  {'name' : os.path.relpath('blinky_tb.v', work_root),
   'file_type' : 'verilogSource'},
  {'name' : os.path.relpath('vlog_tb_utils.v', work_root),
   'file_type' : 'verilogSource'}
]
```

`clk_freq_hz` 디자인에는 정수를 허용 하는 이름을 가진 최상위 Verilog 매개변수가 있습니다. 기본값을 로 설정합니다 `1000`. 테스트 벤치에는 다음과 같은 plusarg를 설정하여 파형 덤핑을 활성화하는 옵션도 있습니다 `vcd`.

```
parameters = {'clk_freq_hz' : {'datatype' : 'int', 'default' : 1000, 'paramtype' : 'vlogparam'},
              'vcd' : {'datatype' : 'bool', 'paramtype' : 'plusarg'}}

```

시뮬레이션에 Icarus Verilog를 사용할 계획임을 Edalize에 알리십시오.

```
tool = 'icarus'
```

그리고 프로젝트의 최상위 및 이름에 대한 몇 가지 정보와 함께 단일 데이터 구조에 모두 넣습니다.

```
edam = {
  'files'        : files,
  'name'         : 'blinky_project',
  'parameters'   : parameters,
  'toplevel'     : 'blinky_tb'
}
```

이제 Edalize에서 백엔드 객체를 가져와야 합니다.

```
backend = get_edatool(tool)(edam=edam,
                            work_root=work_root)
```

디렉터리와 프로젝트 파일을 만듭니다.

```
os.makedirs(work_root)
backend.configure()
```

이 시점에서 우리는 아직 실제 EDA 도구를 실행 `work_root`하지 않았으며 선호하는 경우 Edalize 없이 디렉토리의 파일을 사용할 수 있습니다. 그러나 Edalize로 예제를 계속 진행해 보겠습니다.

시뮬레이션 모델 구축:

```
backend.build()
```

그리고 마지막으로 우리의 주장과 함께 그것을 실행하십시오. 일부 유형의 매개변수(예: plusargs)는 런타임에 정의되며 이 시점에서 이름과 새 값을 에 전달하여 해당 값을 변경할 수 있습니다 `run()`. 또는 완전히 건너뛸 수 있으며 구성 단계의 기본값이 사용됩니다. VCD 로깅을 활성화한 상태에서 실행해 보겠습니다.

```
args = {'vcd' : True}
backend.run(args)
```

타다! 우리는 시뮬레이션했습니다. 연습으로 도구 변수를 예를 들어 modelsim, xsim 또는 Edalize에서 지원하는 다른 시뮬레이터로 변경하고 변경 없이 작동하는지 확인하십시오.

이제 FPGA 이미지를 대신 생성할 차례입니다.

이미 보았듯이 Edalize는 EDA 도구를 인터페이스하기 위한 수상 경력에 빛나는 도구이므로

**그것을 교육하고, 그것을 비판하지 마십시오! 에달라이즈하면 광고하겠습니다!**

자세한 내용은 소스 코드를 참조하십시오.
