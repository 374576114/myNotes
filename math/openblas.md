BLAS
---
# 1 OpenBLAS
[developer manual](https://github.com/xianyi/OpenBLAS/wiki/Developer-manual)

## 1.1 OpenBLAS �ӿڲ�
��OpenBLAS�ӿڲ��У������ַ�Ϊ�������ͣ��ֱ���level1��3������level3��Ӧ����;�������㣬level2��level1����ά��Խ��Խ�͡�������Щlevel1~3����BLAS�ڲ�����ʱʹ�õĽӿڣ�Դ���������driver/level�£���������û��Ľӿ��ǲ��漰�������ģ�����ӿڻ�������interface�ļ����£� 
[github](https://github.com/xianyi/OpenBLAS/tree/develop/interface)  
ÿһ��Դ�����ļ���Ӧһ�ֲ�������gemmָ������ͨ����˷���General Matrix Multiplication����gemvָ������ͨ���������˷���General Matrix Vector��

[��ϸ˵��](https://www.ibm.com/support/knowledgecenter/SSFHY8_5.3.0/com.ibm.cluster.essl.v5r3.essl100.doc/am5gr_hsgemm.htm)������ΪC���������㣬Z����˫���ȸ�������S�������ȳ���
ȫ�������� https://github.com/xianyi/OpenBLAS/blob/develop/cblas.h  

## 1.2 ����
**�������**
```
void cblas_sgemm(const enum CBLAS_ORDER Order,
	const enum CBLAS_TRANSPOSE TransA, const enum CBLAS_TRANSPOSE TransB, 
	const int M, const int N, const int K, 
	const float alpha, 
	const float  *A, const int lda, 
	const float  *B, const int ldb,
	const float beta, 
	float  *C, const int ldc)
```
C := alpha*op(A)*op(B) + beta*C  
op: ��ת�ã�ת�ã�����  

���� | ֵ
:- | :-
Order          | CblasRowMajor, CblasColMajor
TransA, TransB | CblasNoTrans, CblasTrans
M              | op(A) �� C ������
N              | op(B) �� C ������
K              | op(A) �������� op(B) ������


# 2 Caffe �� BLAS ��ʹ��
�������㺯�����ļ�math_functions.hpp�п����ҵ������еĺ������Ƕ�BLAS��ӦAPI�İ�װ  
[�ο�](http://blog.csdn.net/seven_first/article/details/47378697)

## 2.1 ��������󣬾����������ĳ˷�
```
template<>
void caffe_cpu_gemm<float>(
	const CBLAS_TRANSPOSE TransA, const CBLAS_TRANSPOSE TransB, 
	const int M, const int N, const int K,
    const float alpha, const float* A, const float* B, 
	const float beta, float* C)
{
  int lda = (TransA == CblasNoTrans) ? K : M;
  int ldb = (TransB == CblasNoTrans) ? N : K;
  cblas_sgemm(CblasRowMajor, TransA, TransB, M, N, K, alpha, A, lda, B,
      ldb, beta, C, N);
}
```
[�ο�](https://xmfbit.github.io/2017/03/08/mathfunctions-in-caffe/)  

## 2.2 ����/�����ļӼ�
- caffe_axpy(N, alpha, x, mutable y)        �����ӷ� Y = alpha * X + Y
- caffe_axpby(N, alpha, x, beta, mutable y) �����ӷ� Y = alpha * X + beta * Y

## 2.3 �ڴ����

