
---
title: python: cpu & memory & disk 체크
categories:
  - python

tags:
  - cpu
  - memory
  - disk

toc: true
toc_sticky: true
toc_label: Contents
---

일반적으로 CPU나 Memory는 작업관리자나, 커맨드 혹은 터미널 명령어를 통해 조회할 수 있지만,

파이썬의 `psutils` 모듈을 활용하면 편하게 현재 PC의 메모리와 CPU를 체크할 수 있습니다.

> psutils : https://psutil.readthedocs.io/
> 

```python
import psutil
```

## 메모리

메모리 조회는 PC 단위로 확인하는 방법과, 프로세스 단위로 확인하는 방법이 있습니다.

### 1. PC 단위(전체 프로세스)

```python
memory = psutil.virtual_memory()
```

`virtual_memory()` 호출을 통해 메모리에 관련한 다양한 값을 얻을 수 있습니다.

출력을 해보면 아래와 같습니다.

```python
svmem(total=17171070976, available=6445760512, percent=62.5, used=10725310464, free=6445760512)
```

`svmem`은 service memory의 약어로, 반환시 해당 객체로 전달됩니다.

- *total:* 해당 PC의 총 물리적 메모리 값
- *available*: 스왑되지 않은, 가용 가능한 메모리 값
- *percent*: (total - available) / total * 100 으로 계산된 값
- *used*: 사용된 메모리. 하지만 플랫폼에 따라 다르게 계산되고 , total - free = used가 반드시 성립하진 않는다. (정보 제공 목적으로만 사용)
- *free*: 메모리가 사용되지 않은 값. 그러나 사용 가능한 메모리로 취급되지 않는다.

> 플랫폼 별로 제공되는 메모리 통계 항목이 다르며, 위의 값은 window 기준입니다.
> 

퍼센티지를 제외한 절대 값들은 바이트(Byte) 단위이므로, 보기 편한 기가바이트(GB) 혹은 메가바이트(MB) 값으로 변환하려면 계산을 통해 변환해주면 됩니다.

```python
def byte_to_gb(byte):
    return byte / 2**30

def entry_memory():
    memory = psutil.virtual_memory()
    total_memory = byte_to_gb(memory.used)
    
    print(f'Entry Memory: {memory.percent}%, {total_memory:10.3f}GB')
    return memory
```

### 2. 프로세스 단위(단일 프로세스)

프로세스 단위의 메모리 조회는 pid를 가진 OS 프로세스를 대상으로 합니다.

```python
process = psutil.Process(pid)
```

pid가 생략되면, 현재 프로세스 pid를 가져옵니다(`os.getpid`)

> ex) PyCharm에서 실행시, Pycharm.exe의 pid 조회
> 

`psutil.Process()` 실행하여 출력하면 아래와 같은 값처럼 다양한 값이 출력됩니다.

```python
pmem(rss=2726322176, vms=3319754752, num_page_faults=5643692, peak_wset=4399738880, wset=2726322176, peak_paged_pool=7800440, paged_pool=5948728, peak_nonpaged_pool=419736, nonpaged_pool=316688, pagefile=3319754752, peak_pagefile=4161921024, private=3319754752)
```

`pmem`값은 Process Memory의 약어로서, 반환시 해당 객체로 전달됩니다.

많은 값들이 있지만 플랫폼 상관없이 사용할 수 있는 필드는 **rss** 값과, **vms** 값 입니다.

- *rss*: **Resident Set Size**라고 하며, 스왑되지 않은 프로세스가 사용한 실제 메모리 값. (Windows 환경에서는 `wset`과 동일)
- *vms*: **Virtual Memory Size**라고 하며, 프로세스에서 사용하는 총 가상 메모리 값.

`process` 객체로는 필드에 직접적으로 접근할 수 없습니다. 대신 `memory_info()` 함수를 통해 튜플 형태로 받아 필드에 접근할 수 있습니다

`rss`값이 실제 메모리 값을 나타내므로, 해당 필드를 통해 프로세스의 메모리 사용값을 확인할 수 있습니다.

```python
def byte_to_mb(byte):
    return byte / 2**20

def process_memory(pid):
	process = psutil.Process(pid)
	memory_info = process.memory_info()  # process 객체는 필드에 직접적으로 접근할 수 없음
	uesd_memory = byte_to_mb(memory_info.rss)	
	
	print(f'Process Used Memory: {used_memory}')
	return used_memory
```

프로세스 메모리 점유율(퍼센티지)은 계산을 통해서 할 수도 있지만, `process`객체의 `memory_percent()` 함수로 제공하여 이를 사용할 수도 있습니다.

```python
def process_memory_percent(pid):
	process = psutil.Process(pid)
	memory_info = process.memory_info()

	# 1. 전체 메모리 값을 이용하여 계산하기
	total_memory = psutil.virtual_memory().total
	calc_memory_percent = (memory_info.rss / total_memory * 100)
	
	# 2. memory_percent() 함수 이용하기
	func_memory_percent = process.memory_percent()
```

---

## CPU

CPU에 대한 값은, 어떠한 절대값으로 표시되지 않고, 시간이나 퍼센티지로만 제공됩니다.

### 1. PC 단위 (전체 프로세스)

```python
entry_cpu_percent = psutil.cpu_percent()
```

`cpu_percent()`함수는, 시스템 전체 CPU 사용률을 백분율로 나타내는 소수점을 반환합니다.

또한, 호출 파라미터는 2개가 있는데, 아래와 같습니다

- *interval*: CPU 사용률을 계산하는데 사용되는 시간 간격. 짧은 간격으로 지정하면 지나치게 변동적일 수 있으므로 주의해야 한다.
    - interval > 0: block으로 동작
    - interval = 0: non-block으로 동작
    - interval < 0: 에러
- percpu: 각 CPU 코어의 사용률을 반환할 지 여부.

cpu_percent() 함수와 비슷하지만 추가적으로, 특정 CPU 시간에 대한 사용률을 제공하는 함수가 있는데, 바로 `cpu_times_percent()`입니다.

```python
entry_cpu_times_percent = psutil.cpu_times_percent()
```

함수 파라미터는 `cpu_percent()`와 동일하며, `scputimes` 객체를 반환합니다.

해당 객체는, 여러 필드를 가지고 있는데, Windows 플랫폼 기준으로는 아래와 같습니다.

~~근데 사실, 이것까지 확인하진 않을 듯 싶습니다.. (참고만)~~

- user: 사용자 모드에서 실행되는 일반 프로세스에 소요된 시간
- system: 커널 모드에서 실행되는 프로세스에 소요된 시간
- idle: 아무것도 하지 않고 소요된 시간 (=유휴 시간)
- interrupt: 하드웨어 인터럽트 서비스에 소요된 시간
- dpc: DPC 서비스에 소요된 시간 (DPC는 표준 인터럽트보다 낮은 우선순위로 실행되는 인터럽트)

> DPC (Deferred Procedure Call):  지연 프로세스 호출
> 

추가적으로, 코어수나 주파수 값 등을 통계로 내어주는 함수들도 있지만 해당 값들은 주기적인 통계 지표로 사용될 값들은 아니라고 생각해서 제외하였습니다.

### 2. 프로세스 단위 (단일 프로세스)

프로세스 단위의 CPU 측정은 오히려 심플합니다. 

`Process` 객체의 `cpu_percent()` 를 이용하면 됩니다.

```python
process = psutil.Process(pid)
process.cpu_percent()
```

`cpu_percent()`함수는 동일하게 `interval`과 `percpu` 파라미터가 존재합니다.

의미와 사용은 위를 참고하시면 됩니다.

---

## 디스크

```python
entry_disk_usage = psutil.disk_usage(path='/')
```

위의 entry_disk_usage 값은 전체, 사용, 여유, 퍼센트 값을 가지는 `sdiskusage` 객체를 반환합니다

```python
sdiskusage(total=75159826432, used=74029387776, free=1130438656, percent=98.5)
```

각 필드의 의미는 명확하므로 따로 설명하지 않고, 

퍼센트를 제외한, 바이트 단위를 기가바이트로 변환하여 사용하면 됩니다.

```python
def byte_to_gb(byte):
    return byte / 2**30

def entry_disk():
    entry_disk_usage = psutil.disk_usage('/')
    
    total = entry_disk_usage.total
    used = entry_disk_usage.used
    free = entry_disk_usage.free
    percent = entry_disk_usage.percent
    print(f'Entry Disk total: {byte_to_gb(total):10.3f}GB, used: {byte_to_gb(used):10.3f}GB, free:{byte_to_gb(free):10.3f}GB, percent: {percent}%')
```

