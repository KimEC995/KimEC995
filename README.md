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
#include <stdio.h>

__global__ void printData(int* _dDataPtr)
{
	printf("%d", _dDataPtr[threadIdx.x]);
}

__global__ void setData(int* _dDataPtr)
{
	_dDataPtr[threadIdx.x] = 2;
}

int main(void)
{
	int data[10] = { 0 };
	for (int i = 0; i < 10; i++) data[i] = 1;

	int* dDataPtr;

	// 메모리 공간 할당. dDataPtr int 10만큼
	cudaMalloc(&dDataPtr, sizeof(int) * 10);

	// 초기화
	cudaMemset(dDataPtr, 0, sizeof(int) * 10);

	printf("Data in device:");
	// 데이터 출력. 1개 블럭(?) 스레드 10개만큼 -> dDataPtr
	printData <<<1, 10 >>> (dDataPtr);

	// 메모리 복사. data(Host) -> dDataPtr(Device) 를 int 10만큼
	cudaMemcpy(dDataPtr, data, sizeof(int) * 10, cudaMemcpyHostToDevice);
	printf("\nHost -> Device: ");
	printData << <1, 10 >> > (dDataPtr);

	setData << <1, 10 >>>(dDataPtr);
	
	// 메모리 복사.
	cudaMemcpy(data, dDataPtr, sizeof(int) * 10, cudaMemcpyDeviceToHost);
	printf("\nDevice -> Host: ");
	for (int i = 0; i < 10; i++) printf("%d", data[i]);

	cudaFree(dDataPtr);
}

Data in device : 0000000000
Host->Device : 1111111111
Device->Host : 2222222222

-->
