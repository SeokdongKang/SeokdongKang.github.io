---
layout: single
title: strassen algorithm
date: 2020-04-07 08:19:02 +0900
---



- strassen algorithm 이란? 독일의 수학자 쉬트라센이 1969년에 만든 행렬 곱셈 알고리즘으로 기존 무식한 방식에 비해 시간복잡도가 더 낮은 O(n^2.81)을 보여준다.
- 행렬 C는 행렬 A와 B의 연산으로 이루어지며 행렬 A와 B는 2n * 2n 의 크기를 지닌 다. (n은 임의의 정수이다.) 만약 2n * 2n이 아닐 경우 빈 자리를 0으로 채워 2n * 2n 꼴로 만들고 쉬트라센 행렬 곱을 진행한다.
- 따라서 행렬 A, B, C는 서로 크기가 같은 4개의 부분 행렬로 분할이 가능하다.

![img](http://yimoyimo.tk/images/strassen2.png) 

- A, B 행렬을 각가 4개씩 총 8개의 부분행렬로 분할한 후 쉬트라센 알고리즘 연산을 수행한다. 기존에 행렬곱은 단순히 8번의 곱셈과 4번의 덧셈이 필요했지만, 쉬트라센의 방법은 7번의 곱셈과 18번의 덧셈/뺄셈을 필요로 한다. 컴퓨터의 입장에서는 곱셈 연산이 덧셈 뺄셈 연산보다 부담이 크므로 쉬트라센 알고리즘이 훨씬 좋은 알고리즘이다.
- strassen algorithm에 따라 다음 7개의 수식으로 곱 C를 표현할 수 있다.

![img](http://yimoyimo.tk/images/strassen3.png) 

![img](http://yimoyimo.tk/images/strassen5.png) 

### 쉬트라센 알고리즘 구현 소스코드

- [코드 다운로드](https://github.com/yimok/yimok.github.io/tree/master/data/strassen)

```
#include <stdio.h>
#include <stdlib.h>
#include <vector> //vector를 쓰기위한 라이브러리
#include <math.h> //log,floor 사용
#include <time.h> //clock()
using std::vector;

#pragma warning(disable:4996)
#define SIZE 1024 //행렬 사이즈

// 바이너리 파일을 읽어와 벡터 2차원 배열에 저장시키는 함수.
// parameter : 읽은 값을 저장시킬 vector 주소 , 파일명, , 행사이즈, 열사이즈 
// return : 없음
void MatrixInit(vector< vector<int> > &matrix, char * fname, int Row, int Col)
{
	FILE *fin;
	fin = fopen(fname, "rb");
	char * arr = new char[Row*Col];
	int idx = 0;

	if (fin == NULL)
	{
		printf("파일명이 없습니다.\n");
		return;
	}

	if (fread(arr, Row*Col, 1, fin) != 1)
	{
		printf("에러");
		exit(1);
	}


	for (int i = 0; i <= Row - 1; i++)
	{
		for (int j = 0; j <= Col - 1; j++)
		{
			matrix[i][j] =  arr[idx];
			idx++;
		}

	}
	fclose(fin);

}


// 행렬 A와 B를 더하여 C에 저장시키는 함수(A행렬 B행렬 C행렬은 같은 크기)
// parameter :행렬 A,B,C 
// return : 없음
void MatrixSum(vector< vector<int> > &matrixA, vector< vector<int> > &matrixB, vector< vector<int> > &matrixC)
{

	for (int i = 0; i < (int)matrixA.size(); i++) // matrixA.size() = 행 길이
	{
		for (int j = 0; j< (int)matrixA[i].size(); j++) // matrixA.size() = 열 길이
		{
			matrixC[i][j] = matrixA[i][j] + matrixB[i][j];
		}
	}

}

// 행렬 A와 B를 빼서 C에 저장시키는 함수(A행렬 B행렬 C행렬은 같은 크기)
// parameter : 행렬 A,B,C 
// return : 없음
void MatrixSub(vector< vector<int> > &matrixA, vector< vector<int> > &matrixB, vector< vector<int> > &matrixC)
{
	for (int i = 0; i < (int)matrixA.size(); i++) // matrixA.size() = 행 길이
	{
		for (int j = 0; j< (int)matrixA[i].size(); j++) // matrixA.size() = 열 길이
		{
			matrixC[i][j] = matrixA[i][j] - matrixB[i][j];
		}
	}


}

// 행렬 A와 B를 곱하여 C에 저장시키는 함수(A행렬 B행렬 C행렬은 같은 크기)
// parameter : 행렬 A, B, C 
// return : 없음
void MatrixMul(vector< vector<int> > &matrixA, vector< vector<int> > &matrixB, vector< vector<int> > &matrixC)
{
	for (int i = 0; i < (int)matrixA.size(); i++) // matrixA.size() = 행 길이
	{
		for (int j = 0; j< (int)matrixA[i].size(); j++) // matrixA.size() = 열 길이
		{
			for (int k = 0; k < (int)matrixA[i].size(); k++)
			{
				matrixC[i][j] += matrixA[i][k] * matrixB[k][j];
			}
		}
	}


}





// 임계값 구하는 함수
// parameter : 행렬의 가로 및 세로 길이 (ex:1024)
// return : 임계값 반환
int getThreshold(int n)
{
	int th;
	double k = floor(log(n) / log(2) - 6);
	th = (int)floor(n / pow(2.0, k)) + 1;
	return th;
}


// 4개의 부분행렬로 나누는 함수
// parameter : 나눌 행렬, 저장할 행렬 공간 4개
// return : 없음
void Submatrix(vector< vector<int> > &matrixOrigin, vector< vector<int> > &matrix11, vector< vector<int> > &matrix12,
	vector< vector<int> > &matrix21, vector< vector<int> > &matrix22)
{
	int newNum = matrix11.size();  //부분행렬의 사이즈 

	for (int i = 0; i < newNum; i++)
	{
		for (int j = 0; j < newNum; j++)
		{
			matrix11[i][j] = matrixOrigin[i][j];						//좌 상단행렬
			matrix12[i][j] = matrixOrigin[i][j + newNum];				//우 상단행렬
			matrix21[i][j] = matrixOrigin[i + newNum][j];				//좌 하단행렬
			matrix22[i][j] = matrixOrigin[i + newNum][j + newNum];		//우 하단행렬
		}
	}

}


// 4개의 부분행렬들을 재결합 해주는 함수
// parameter : 합친 결과를 저장할 행렬 , 부분행렬 11, 12, 21, 22
// return : 없음
void Mergematrix(vector< vector<int> > &matrixOrigin, vector< vector<int> > &matrix11, vector< vector<int> > &matrix12,
	vector< vector<int> > &matrix21, vector< vector<int> > &matrix22)
{
	int newNum = matrix11.size();  //부분행렬의 사이즈

	for (int i = 0; i < newNum; i++)
	{
		for (int j = 0; j < newNum; j++)
		{
			matrixOrigin[i][j] = matrix11[i][j];						//좌 상단행렬
			matrixOrigin[i][j + newNum] = matrix12[i][j];				//우 상단행렬
			matrixOrigin[i + newNum][j] = matrix21[i][j];         	    //좌 하단행렬
			matrixOrigin[i + newNum][j + newNum] = matrix22[i][j];	    //우 하단행렬
		}
	}


}





// 쉬트라센 알고리즘 함수
// parameter : 행렬의 사이즈(ex: 1024x1024 -> 1024입력) , 행렬 A, B, C
// return : 없음
void Strassen(int n, vector< vector<int> > &matrixA, vector< vector<int> > &matrixB, vector< vector<int> > &matrixC)
{


	if (n <= getThreshold(n))
	{
		MatrixMul(matrixA, matrixB, matrixC);
		return;
	}
	else
	{
		int newRow = n / 2;					//4등분을 하기 위해
		vector<int> newCol(newRow, 0);

		//a11~a22 부분행렬, b11~b22 부분행렬 
		vector < vector<int> > a11(newRow, newCol), a12(newRow, newCol), a21(newRow, newCol), a22(newRow, newCol);
		vector < vector<int> > b11(newRow, newCol), b12(newRow, newCol), b21(newRow, newCol), b22(newRow, newCol);

		//부분행렬들의 연산결과를 m1~m7 에저장
		vector < vector<int> > m1(newRow, newCol), m2(newRow, newCol), m3(newRow, newCol), m4(newRow, newCol)
			, m5(newRow, newCol), m6(newRow, newCol), m7(newRow, newCol);

		//a11~b22 의 연산결과들을 임시로 저장할 그릇
		vector < vector<int> > tempA(newRow, newCol), tempB(newRow, newCol);

		// m1~m7 연산 결과로 C를 구하기 위해 저장 할 행렬
		vector < vector<int> > c11(newRow, newCol), c12(newRow, newCol), c21(newRow, newCol), c22(newRow, newCol);


		//A의 부분행렬 4개, B의 부분행렬 4개 생성
		Submatrix(matrixA, a11, a12, a21, a22);
		Submatrix(matrixB, b11, b12, b21, b22);


		MatrixSum(a11, a22, tempA);		       // a11+a22
		MatrixSum(b11, b22, tempB);		       // b11+b22
		Strassen(newRow, tempA, tempB, m1);    // m1=(a11+a11)(b11+b22)
		
		MatrixSum(a21, a22, tempA);            // a21+a22
		Strassen(newRow, tempA, b11, m2);      // m2=(a21+a22)b11

		MatrixSub(b12, b22, tempB);            // b12-b22
		Strassen(newRow, a11, tempB, m3);      // m3=a11(b12-b22)

		MatrixSub(b21, b11, tempB);            // b21-b11
		Strassen(newRow, a22, tempB, m4);      // m4=a22(b21-11)

		MatrixSum(a11, a12, tempA);            //  a11+a12
		Strassen(newRow, tempA, b22, m5); 	   // m5=(a11+a12)b22

		MatrixSub(a21, a11, tempA);            // a21-a11
		MatrixSum(b11, b12, tempB);            // b11+b12
		Strassen(newRow, tempA, tempB, m6);    // m6=(a21-a11)(b11+b12)

		MatrixSub(a12, a22, tempA);            // a12-a22
		MatrixSum(b21, b22, tempB);            // b21+b22
		Strassen(newRow, tempA, tempB, m7);    // m7 = (a12 - a22)(a12 - a22)




		// 위에서 계산된 m1~m7 결과로  c11 ~ c22 를 만든다.
		MatrixSum(m1, m4, tempA); //m1 + m4
		MatrixSum(tempA, m7, tempB); //m1 + m4 + m7
		MatrixSub(tempB, m5, c11); //c11 = m1 + m4 - m5 + m7

		MatrixSum(m3, m5, c12); //c12 = m3 + m5

		MatrixSum(m2, m4, c21); //c21 = m2 + m4

		MatrixSum(m1, m3, tempA); //m1 + m3
		MatrixSum(tempA, m6, tempB); //m1 + m3 + m6
		MatrixSub(tempB, m2, c22); //c22 = m1 + m3 - m2 + m6



		//재 병합
		Mergematrix(matrixC, c11, c12, c21, c22);


	}
}


void FileOutput(char *fname, vector< vector<int> > &matrix)
{
	FILE *fout;
	fout = fopen(fname, "wt");   

	for (int i = 0; i < (int)matrix.size(); i++)
	{
		for (int j = 0; j < (int)matrix.size(); j++)
		{
			fprintf(fout, "%d  ", matrix[i][j]);
		}
		fprintf(fout, "\n");
	}

}



int main()
{
	int Row = SIZE;
	int Col = SIZE;
	char fname1[50], fname2[50];
	int start, end; //시간측정


	// ex) vector<vector<int> >  arr(6, vector<int>(5, 0));
	// 설명: vector< vector<int> > 형 벡터 6개(가로 6줄)를 할당 한다는 뜻
	//       vector<int>(5,0) 은 모든 가로줄은 5개짜리 0으로 초기화 된 익명의 int 형 벡터 배열을 생성해 초기값을 넣는다.

	vector < vector<int> > A(Row, vector<int>(Col, 0)); // A[Row][Col] 인 동적 배열
	vector < vector<int> > B(Row, vector<int>(Col, 0)); // B[Row][Col] 인 동적 배열
	vector < vector<int> > C(Row, vector<int>(Col, 0)); // C[Row][Col] 인 동적 배열
	vector < vector<int> > D(Row, vector<int>(Col, 0)); // D[Row][Col] 인 동적 배열

	printf("A행렬에 입력할 파일명을 입력하세요 \n( 입력 예 : xxx.txt )");
	scanf("%s", &fname1);
	printf("B행렬에 입력할 파일명을 입력하세요 \n( 입력 예 : xxx.txt )");
	scanf("%s", &fname2);


	MatrixInit(A, fname1, Row, Col);
	MatrixInit(B, fname2, Row, Col);

	start = clock();
	MatrixMul(A, B, D);
	end = clock();
	printf("Navie matrix multiplication time : %f ms \n", (double)(end - start));

	start = clock();
	Strassen(A.size(), A, B, C);
	end = clock();
	printf("Strassen matrix multiplication time : %f ms \n", (double)(end - start));

	FileOutput("Navie result.txt", D);
	FileOutput("Strassen result.txt", C);
	printf("무식한 방법 결과가 Navie result.txt에 저장되었습니다. \nStrassen 방법이 Strassen result.txt에 저장되었습니다.\n\n");
	

	return 0;
}
```

### 코드 해설

1. MatrixInit 함수 : 인자로 파일에서 읽은 값을 저장시킬 vector 2차원 배열, 불러올 파일명, 행사이즈, 열사이즈를 입력받는다. 함수 내부에 FILE 포인터 변수 fin을 선언하고 바이너리 파일을 읽을 수 있도록 rb로 파일을 오픈한다. 읽어온 값들을 1차적으로 저장 시키기 위해 동적배열 arr을 만들어 준 후 fread(arr, Row x Col , 1 , fin) 함수를 호출한다. 즉 1024 x 1024 개수의 데이터를 1바이트씩 값을 읽어서 arr 배열에 저장 시킨다. 1차적으로 arr 배열에 저장한 데이터들을 vector 2차원 배열에 1024*1024 행렬로 저장 시킨다.
2. MatrixSum 함수 : 인자로 vector 2차원 배열 3개를 입력 받는다. 이 3개의 배열은 각각 행렬A, 행렬B, 행렬C 로 A+B=C 의 동작을 수행한다. 2중 for문을 사용하여 i변수와 j변수를 행, 열로 활용하여 각 행렬의 원소 값 들을 연산한다. for문의 최대 반복수는 matrixA.size() 즉 인자로 입력받은 행렬의 사이즈 이다.(1024 x 1024 이면 size는 1024) 정사각 행렬이기 때문에 matrixA[i].size() 도 역시 같다.
3. MatrixSum 함수 : 인자로 vector 2차원 배열 3개를 입력 받는다. 이 3개의 배열은 각각 행렬A, 행렬B, 행렬C 로 A-B=C 의 동작을 수행한다. 2중 for문을 사용하여 i변수와 j변수를 행, 열로 활용하여 각 행렬의 원소 값 들을 연산한다. for문의 최대 반복수는 matrixA.size() 즉 인자로 입력받은 행렬의 사이즈 이다.(1024 x 1024 이면 size는 1024)  정사각 행렬이기 때문에 matrixA[i].size() 도 역시 같다.
4. MatrixMul 함수 : 인자로 vector 2차원 배열 3개를 입력 받는다. 이 3개의 배열은 각각 행렬A, 행렬B, 행렬C 로 A x B = C 의 동작을 수행한다. 3중 for문을 사용하여 i변수와 j변수k변수를 생성하여 일반적인 행렬의 곱셈 방식을 구현한다. 즉 행렬의  원소 위치는  이므로 k를 유동적으로 변동시켜서 무식한 방법의 행렬 곱을 구현한다. for문의 최대 반복수는 matrixA.size() 즉 인자로 입력받은 행렬의 사이즈 이다. (1024*1024 이면 size는 1024)  정사각 행렬이기 때문에 matrixA[i].size() 도 역시 같다.
5. getThreshold 함수 : 임계값을 구하는 함수로 쉬트라센 알고리즘에서 입력된 크기 n이 어느 임계값을 넘지 못한다면 무식한 방법의 행렬 곱을 사용하는 것이 더 효과적이다. 이 임계값을 구하기 위한 함수이다.
6. Submatrix 함수 : 인자로 나눌 행렬 원본, 저장할 행렬 공간 4개를 입력받는다. 원본 행렬을 4개의 부분행렬로 나누어야하므로 부분행렬의 size 만큼 for문 2개를 구성하여 원래행렬을 인덱스 i, j 에 newNum을 각 위치에 맞게 변동 시키면서 각각 좌 상단행렬, 우 상단행렬, 좌 하단행렬, 우 하단 행렬에 적재 시킨다.
7. Mergematrix 함수 : 인자로 합친 결과를 저장할 행렬, 부분행렬 matrix11, matrix12, matrix21, matrix22  4개를 입력받는다. 4개의 부분행렬을 하나로 합쳐야 하므로 부분행렬의 size 만큼 for문 2개를 구성하여 합친 결과를 저장할 행렬의 인덱스 i, j 에 newNum을 각 위치에 맞게 변동 시키면서 각각 좌 상단행렬, 우 상단행렬, 좌 하단행렬, 우 하단 행렬을 matrixOrigin 행렬에 하나로 합친다.
8. Strassen 함수 : 인자로 쉬트라센 알고리즘을 사용할 행렬의 사이즈(ex: 1024 x 1024 -> 1024), 3개의 Vector 2차원 배열(행렬 A, B, C)을 입력받는다. 우산 getThreshold 함수로 받아온 행렬의 사이즈가 임계값보다 큰지 확인을 하여 만약 넘지 못한다면 일반적인 무식한 방법으로 행렬의 곱셈을 수행한다. 넘는다면 받아온 사이즈를 반으로 쪼갠 후 newRow 변수에 넣고 이 크기에 맞게 Vector 2차원 배열들을 생성해 준다. (부분행렬을 저장 할 a11~b22 / 부분행렬들의 연산결과를 저장 할 m1~m7 / a11~b22의 연산결과를 임시로 저장할 그릇 tempA, tempB / m1~m7의 연산결과로 C를 구하기 위해 저장 할 행렬 c11~c22)
9. 그리고 Submatrix 함수를 호출하여 행렬A와 행렬B를 각각 4개의 부분행렬로 나누어 a11~b22 vecotr 2차원 배열들에 저장시킨다. 그런 후 쉬트라센 공식을 진행한다. 각 공식에 맞게 +와 – 연산은 MatrixSum, MatrixSub 함수로 구현하고 곱셈부분은 재귀적으로 Strassen 함수를 호출 하여 구현한다. 즉 Strassen 함수에 MatrixSum, MatrixSub 의 결과와 m1~m7을 인자로 넣어주어 곱셈을 실행한다. 이렇게 재귀적으로 구현하면 n이 임계값보다 작아질 때까지 부분행렬로 쪼개지다가 마지막에 크기가 작아진 상태에서 MatrixMul 함수를 호출하여 일반적인 행렬 곱을 수행한다.  이 부분을 분할 정복 기법으로 볼 수 있다.
10. 그렇게 수행된 결과를 통하여 m1~m7을 얻을 수 있고 다시 이 값들을 더하여 c11=m1+m4-m5+m7 , c12 = m3+m5 , c21 = m2+m4 , c22 = m1+m3-m2+m6을 저장 시키고 c11,c12,c21,c22와 저장시킨 행렬C를 매개변수로 넣어 Megematrix 함수를 호출하여 구성한다. 재귀적으로 구현하였기 때문에 분할 될 때까지 분할된 후 일반적인 행렬 곱이 수행 된 다음 이 부분을 통하여 재 병합이 된다.
11. FileOutput 함수 : 인자로 저장시킬 파일명, 저장할 vector 2차원 배열을 입력받으며 2중 for문으로 vector 2차원 배열의 데이터를 3칸씩 공백과 개행을 주면서 1024 * 1024 로 텍스트 파일에 저장시킨다.

### 코드 실행 결과

![img](http://yimoyimo.tk/images/strassen.png) 

- 추가적으로 .. vector를 사용하니깐 굉장히 시간이 오래걸렸다 단순히 동저배열을 사용하면 6초가 걸린다. 벡터에 대한 이해가 부족한것도 있고 슈트라센 같은경우 배열을 나누고 다시 합치는 작업을 하는데 이과정에서 벡터가 오버헤드를 많이 잡아먹는것 같다.