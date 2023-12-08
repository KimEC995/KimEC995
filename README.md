## READ ME 👋

`2014 - 2018` 
> 디자인 대학교. 도예 전공
>
> 졸업 논문 : C언어와 도자기의 결합 == 움직이는 도자기(아두이노 활용)

`2017 - 2017`
>VR게임 R&D 팀 작업. // 모션 캡쳐 마커 및 반응형 총기 장난감 개발.
>
>아두이노와 DC 모터 제어 코드를 활용한 충격 전달 총기 장난감.

`2017 - 2022`
>메이커스페이스 // 중기청 국책사업 기획, 운영, 개발, 관리, 교육
>
>시제품 개발 회사 // 시제품 3D 설계, 개발, C언어를 활용한 아두이노 적용

`2023.06 - 2023.11`
> 국비교육: 멀티캠퍼스 - 데이터엔지니어 & 분석가(23.06~23.11)

`2023.11 - `
> 멀티미디어 R&D // C++과 CUDA를 활용한 다중 멀티미디어 병합 제어


`사이드 프로젝트`
- PETE 필라멘트 재활용기
> `C++` Timer 셋을 활용한 함수 병렬 사용/ 사용자 반응 수집

- Pi Nas 개인형 나스서버 구축
> `Linux` 서버 구축 및 관리
<!--
**KimEC995/KimEC995** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

#include "cuda_runtime.h"
#include "device_launch_parameters.h"

#include "DS_timer.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

//벡터의 크기는 기호상수(Symbolic Constant) 로 정의.
#define NUM_DATA 134217728

//커널 정의(벡터연산)
__global__ void vecADD(int* _a, int* _b, int* _c, int _size)
{
	//thread ID
	int tID = (blockIdx.x * blockDim.x) + threadIdx.x;

	//연산
	if (tID < _size)
		_c[tID] = _a[tID] + _b[tID];
}

int main(void)
{
	//Set Timer
	DS_timer timer(5);
	timer.setTimerName(0, (char*)"CUDA Total-----------------:");
	timer.setTimerName(1, (char*)"Computation(Kernel)--------:");
	timer.setTimerName(2, (char*)"Data Trans. Host -> Device-:");
	timer.setTimerName(3, (char*)"Data Trans. Device -> Host-:");
	timer.setTimerName(4, (char*)"VecAdd on Host-------------:");

	//타이머 초기화
	timer.initTimers();

	int* a, * b, * c, * hc;    //Host벡터
	int* da, * db, * dc;       //Device 벡터

	int memSize = sizeof(int) * NUM_DATA;
	printf("%d elements, memSize = %d bytes \n", NUM_DATA, memSize);

	// Host단에서 할당, 초기화0
	a = new int[NUM_DATA]; memset(a, 0, memSize);
	b = new int[NUM_DATA]; memset(a, 0, memSize);
	c = new int[NUM_DATA]; memset(a, 0, memSize);
	hc = new int[NUM_DATA]; memset(a, 0, memSize);

	// 난수 할당
	for (int i = 0; i < NUM_DATA; i++)
	{
		a[i] = rand() % 10;
		b[i] = rand() % 10;
	}

	// 비교용: Host단에서 연산해보기(직렬연산)
	timer.onTimer(4);    // VecAdd on Host 타이머 시작
	for (int i = 0; i < NUM_DATA; i++)
	{
		hc[i] = a[i] + b[i];
	}
	timer.offTimer(4);    // VecAdd on Host 타이머 끝

	// Device 메모리 할당, 초기화
	cudaMalloc(&da, memSize); cudaMemset(da, 0, memSize);
	cudaMalloc(&db, memSize); cudaMemset(db, 0, memSize);
	cudaMalloc(&dc, memSize); cudaMemset(dc, 0, memSize);

	timer.onTimer(0);    // CUDA Total 타이머 시작

	// 벡터 복사(Host -> Device)
	timer.onTimer(2);    // Data Trans. Host -> Device 타이머 시작
	cudaMemcpy(da, a, memSize, cudaMemcpyHostToDevice);
	cudaMemcpy(db, b, memSize, cudaMemcpyHostToDevice);
	timer.offTimer(2);    // Data Trans. Host -> Device 타이머 끝

	//스레드 레이아웃 설정
	dim3 dimGrid(ceil((float)NUM_DATA / 256), 1, 1);
	dim3 dimBlock(256, 1, 1);

	// 커널 호출
	timer.onTimer(1);    // Computation(Kernel) 타이머 시작
	vecADD << < dimGrid, dimBlock >> > (da, db, dc, NUM_DATA);
	timer.offTimer(1);    // Computation(Kernel) 타이머 끝

	// 벡터 복사(Device -> Host)
	timer.onTimer(3);    // Data Trans. Device -> Host 타이머 시작
	cudaMemcpy(c, dc, memSize, cudaMemcpyDeviceToHost);
	timer.offTimer(3);    // Data Trans. Device -> Host 타이머 끝

	// Device 메모리 해제
	cudaFree(da);
	cudaFree(db);
	cudaFree(dc);

	timer.offTimer(0);    // CUDA Total 타이머 끝

	// 타이머 출력
	timer.printTimer();

	// 연산 결과 비교
	bool result = true;
	for (int i = 0; i < NUM_DATA; i++)
	{
		if (c[i] != hc[i])
		{
			printf("[%d] Result Not Matched!! (%d, %d)\n", i, c[i], hc[i]);
			result = false;
		}
	}
	if (result)
		printf("GPU Works Well\n");

	// Host 메모리 해제
	delete[] a;
	delete[] b;
	delete[] c;

	return 0;
}

134217728 elements, memSize = 536870912 bytes

* DS_timer Report *
*The number of timer = 5, counter = 5
* ***Timer report * ***
CUDA Total---------------- - : : 295.51350 ms
Computation(Kernel)-------- : : 0.76600 ms
Data Trans.Host->Device - : : 194.89870 ms
Data Trans.Device->Host - : : 86.40610 ms
VecAdd on Host------------ - : : 628.71980 ms
* ***Counter report * ***
*End of the report *
GPU Works Well
-->
