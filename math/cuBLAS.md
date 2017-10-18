#cublas
---
# 1 INTRODUCTION
(�������Դ����ӳ���)��ֵ�����  
[�ٷ�ԭ��](http://docs.nvidia.com/cuda/cublas/index.html#introduction)  

��CUDA6.0֮��cuBLAS ��������API����һ����Ϊ**cuBLAS API**���ڶ�����**CUBLASXT API**  
ʹ��**cuBLAS API** ��Ӧ��һ��Ҫ��GPU���ڴ�ռ��з�����Ҫ�ľ���������������и�ֵ��Ȼ��������õ�cuBLAS������֮���GPU�ڴ��ȡ���ݵ�host�ˡ�cuBLAS API �ṩ�˶�дGPU�ڴ�İ�������   
ʹ��**CUBLASXT API**��Ӧ����Ҫ�����ݷ���HOST��Ȼ��������û���Ҫ�����ݵ��ȵ�GPUS�Ĵ�����  

## 1.1 Data layout
Ϊ����������������е�Fortran ������cuBLAS ���õ��������ȵĴ洢��ʽ�ͻ���1�ļ���  
1  4  7  
2  5  8  
3  6  9  
```
#define IDX2C(i,j,ld) (((j)*(ld))+(i))
```
������ʾ�������i�е�j�е�Ԫ����C������ ����洢λ�õ�������ld��ʾ���� ����ĵ�һά��Ԫ�ظ��������� ���������  
**ʹ�����ֺ�ĳ���������**  
��Ҫ�����ľ�����ΪCUBLAS�⺯��׼���ġ�����˵������׼�������������Ϊ�������ݸ�CUBLAS�Ŀ⺯����
���CUBLAS�еõ���ĳ�����������ʹ���������������Ӧ��Ԫ�ء�  
����ϸ�Ŀ�[����](http://dev.dafan.info/detail/157153?p=30-53)

## 1.2 New and Legacy cuBLAS API
��4.0�汾֮��cuBLAS���ṩ��ͳ��API��ͬʱҲ�ṩ�¸��µ�API������ط����������ṩ��API�������ŵ��Լ���ͳ��API������  
ͨ��ͷ�ļ�"cublas_v2.h"ʹ���µ�API
����������� ��ͳAPIû�еģ� 
- 1 cuBLAS�������ľ���ú�����ʼ�����ұ����ݵ�����ÿһ��ʹ�õĿ⺯������ʹ���û��ڶ������̺߳Ͷ�GPUʱ�Կ�������и���Ŀ���Ȩ����ʹ��cuBLAS API�������루reentrant��
- 2 �������ͦ¿���ͨ�������������豸������Ȼ����д��ݣ��������������ϱ�����Ȼ��ͨ��ֵ���д��ݡ�����ı�ʹ�ÿ⺯��������������ִ��ͬ������ʹ���ͦ� ����ǰ��ĺ˲�����
- 3 ��һ���⺯������һ�����������������ͨ��һ�������������豸�����ı������أ������Ƿ��������е���ֵ�����ʹ�ÿ⺯������ֱ��ʹ�÷���ֵ
- 4
����״̬ cublasStatus_t�����õ���
- 5
the cublasAlloc() and cublasFree() functions have been deprecated. This change removes these unnecessary wrappers around cudaMalloc() and cudaFree(), respectively
- 6
cublasSetKernelStream() ������Ϊ cublasSetStream()

## 1.3 Example code

# 2 Using the cuBLAS API
## 2.1 General description
### 2.1.1 Error status
���е�cuBLAS�⺯��������the error status cublasStatus_t

### 2.1.2 cuBLAS context
Ӧ�ñ���ͨ������cublasCreate()��������ʼ��cuBLAS ��������ľ����ͨ��cublasDestory()���ͷš�Ӧ�ó������ʹ��cudaSetDevice()��ϳ�ʼ���Ķ��������ľ����Ӧ�ó�����ݲ�ͬ�ľ�������ݴ��ݵ���ͬ���豸���м��㡣�����һ����������ʹ�ò�ͬ�����ã���ʹ�����豸ǰ����cudaSetDevice()��Ȼ��cublasCreate()���ݵ�ǰ�������豸����ʼ����ͬ�������ľ����

### 2.1.5 Scalar Parameters
���������ĺ���ʹ�ñ���������  
- 1 �������������豸�ڶ���Ħ����ߦ� ��Ϊ�仯���ӵĺ�������gemm
- 2 �������豸���������ϵı����ĺ������� amax(), amin(), asum(), rotg(), rotmg(), rotmg(), dot() �� nrm2()  

���ڵ�һ����𣬵�ָ��ģʽ����Ϊ��CUBLAS_POINTER_MODE_HOST��,����������������ߦ¿�����ջ���߷����ڶ��С�Underneath the CUDA kernels related tothat functions will be launched with the value of alpha and/or beta. �����������ڶ��б����䣬���ǿ����ڵ��÷Żغ��ͷż�ʹ�ں����첽�ġ���ָ���ģʽ����Ϊ��CUBLAS_POINTER_MODE_DEVICE�������ߦ� Ӧ�����豸�Ͽ��Ա����ʲ��Ҳ��ܱ��޸�ֱ���˵�����ɡ�ע������cudaFree��ʽ����cudaDeviceSynchronize()��cudaFree() can still be called on alpha and/or beta just after the call but it would defeat the purpose of using this pointer mode in that case.  
�ڶ���ģʽ��ָ��ģʽΪ��CUBLAS_POINTER_MODE_HOST����GPU������ɺ����ḳֵ��Host��ָ��ģʽ����Ϊ ��CUBLAS_POINTER_MODE_DEVICE���⺯���������ء�����������;����������������ƣ��������ֻ�е�������GPU����ɡ�Ϊ�˴��������������������Ҫ����ȷ��ͬ���� 
���κ�һ�������ָ��ģʽΪ��CUBLAS_POINTER_MODE_DEVICE���⺯����������ȫ�첽ʱִ�У���ʹalpha �� beta��ǰһ���˲��������磬����cuBLAS��ѭ���ķ����������ϵͳ������ֵ�������ǻ����

## 2.2 cuBLAS Datatypes
### 2.2.1 cublasHandle_t
��cublasCreate() ��ʼ���������������õ�ʱ����

### 2.2.2 cublasStatus_t
���к��������������
����μ�[����](http://docs.nvidia.com/cuda/cublas/#)

### 2.2.7 cublasSetPointerMode_t
һ�κ��������ж������scalar��mode������ͬ
Value | Meaning
:- | :-:
CUBLAS_POINTER_MODE_HOST   | the scalars are passed by reference on the host
CUBLAS_POINTER_MODE_DEVICE | the scalars are passed by reference on the device


## 2.3 CUDA Datatypes

## 2.4 Helper functions
### 2.4.1 cublasCreate()
```
cublasStatus_t cublasCreate(cublasHandle_t *handle)
```
Return Value | Meaning
:- | :-:
CUBLAS_STATUS_SUCCESS         | the initialization succeeded
CUBLAS_STATUS_NOT_INITIALIZED | the CUDA? Runtime initialization failed
CUBLAS_STATUS_ALLOC_FAILED    | the resources could not be allocated  

### 2.4.2 cublasDestroy()
```
cublasStatus_t cublasDestroy(cublasHandle_t handle)
```

### 2.4.8 cublasSetVector()
```
cublasStatus_t cublasSetVector(int n, int elemSize,
		const void *x, int incx, 
		void *y, int incy)
```
��n������ΪelemSize�Ĵ������ڴ��ַ x ���Ƶ� GPU�ڴ��ַy  
incx �� incy Ϊÿ��Ԫ��ͷ�����


### 2.4.9 cublasGetVector()
����ͬ�ϣ���GPU��host

### 2.4.10 cublasSetMatrix()
```
cublasStatus_t cublasSetMatrix(int rows, int cols, int elemSize, 
		const void *A, int lda, void *B, int ldb)
```

### 2.4.11 cublasGetMatrix()

## 2.5 Level-1 functions
scalar and vector based operations
### 2.5.4 axpy()
$y += ��x$  
```
cublasStatus_t cublasSaxpy(cublasHandle_t handle, int n,
			const float *alpha, 
			const float *x, int incx, 
			float *y, int incy)
```

### 2.5.6 dot()
�������
```
cublasStatus_t cublasSaxpy(cublasHandle_t handle, int n,
			const float *x, int incx, 
			const float *y, int incy, 
			float *result)
```

### 2.5.6 scal()
����*����
$x += ��x$  
```
cublasStatus_t cublasSscal(cublasHandle_t handle, int n,
			const float *alpha, 
			float *x, int incx)
```

## 2.6 Level-2 functions

## 2.7 Level-3 functions

$y = �� op ( A ) x + �� y$  
where A is a m �� n matrix stored in column-major format, x and y are vectors, and �� and �� are scalars. Also, for matrix A  
op ( A ) = A  if transa == CUBLAS_OP_N A T  if transa == CUBLAS_OP_T A H  if transa == CUBLAS_OP_H



# �ο�
[����CUBLAS(3)����level2����](http://dev.dafan.info/detail/157153?p=30-53)