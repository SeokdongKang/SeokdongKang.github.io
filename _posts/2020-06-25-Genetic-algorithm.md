---
layout: single
title: "Genetic algorithm"
date: 2020-06-25 20:19:02 +0900
---



import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.Random;

//y=(b/a)x+k 에서 처음 그래프 f(x)에서의 570과 k값이 가장 적은 차이를 나타낼떄  k,x값을 찾는 코드
public class genetic {
	// 00년도 ~ 18년도 , 배열안의 i값은 년도

    public static int fx(int x) {
        return -(2/13)*x+570; //0번째 점과 18번째 점을 이은 일차식 (비교대상)
    }
    public static int[] gene(int[] arr) 
    {
    	int a;
    	Random r = new Random();
    	int[] x = new int[8];
    	for(int i=0;i<3;i++) //3번만 돌리는 이유는 원래의 자료에 대한 값이 남아 있어야 나중에 차이가 적은 k값을 찾을때 0이나올 확률이 작기 떄문에.
    	{
    		a = r.nextInt(8+1);
    		x[a] = (arr[i]+arr[a])/2; //i번쨰 x값과 무작위번쨰 x값의 합의 2분의1
    	}
    	return x;
    }
    public static int[] mutation(int[] x) {
        int[] arr = new int[x.length];
        for (int i=0; i<x.length; i++) {
            arr[i] = arr[i] + arr[x.length];
        }
        return arr;
    }
    
    public static void main(String[] args) {
    	//신생아수 단math.abs위는 천명
    	int x[] = new int[19];
    	x[0] = 635;
    	x[1] = 555;
    	x[2] = 492;
    	x[3] = 491;
    	x[4] = 473;
    	x[5] = 435;
    	x[6] = 448;
    	x[7] = 493;
    	x[8] = 466;
    	x[9] = 445;
    	x[10] = 470;
    	x[11] = 471;
    	x[12] = 485;
    	x[13] = 436;
    	x[14] = 435;
    	x[15] = 438;
    	x[16] = 406;
    	x[17] = 358;
    	x[18] = 327;
    	//년도별 인구수 , 단위는 10만명 , 만명 단위는 내림
    		int y[] = new int[19];
    		y[0] = 470;
    		y[1] = 473;
    		y[2] = 476;
    		y[3] = 478;
    		y[4] = 480;
    		y[5] = 481;
    		y[6] = 484;
    		y[7] = 486;
    		y[8] = 490;
    		y[9] = 493;
    		y[10] = 495;
    		y[11] = 499;
    		y[12] = 501;
    		y[13] = 504;
    		y[14] = 507;
    		y[15] = 510;
    		y[16] = 512;
    		y[17] = 513;
    		y[18] = 516;
    		Random r = new Random();
        	int f1,f2;   
        	int min = 1000;
            int savex=0;
            int arr[] = new int[8];
            for(int j=0;j<3;j++) 
            {
            for(int i=0; i<8; i++) 
            {
            	f1 = r.nextInt(18+1);
            	f2 = r.nextInt(18+1);
                int a,b,c,k;
                a=x[f1]-x[f2];	// y=(b/a)x+k
                b=y[f1]-y[f2];//        b/a=기울기 , b/a * x[f1] -y[f1] = 상수k
                k=(b/a)*x[f1]-y[f1];
              //어차피 무작위 두점을 고르더라도 처음의 직선의방정식의 기울기와 오차가 1까지도 안나기 때문에 기울기를 비교하는것이 아닌 상수를 비교하는게 값을 보기 편하다.
            	c=Math.abs(570-k);
            	if(min>c)
            	{
            		min = c; 
            	}
                arr[i] = f1;
                savex=f1;
                System.out.printf("%d ,%d", c,f1);
                System.out.println();
            } //변형되고 나서는 x값이 원래의 자료에 없는 값이 나올수 있기 때문에 원래 f(x)에 대입을 하여 y값을 구한다.
            gene(arr);
            }
            	System.out.printf("가장 작은 k의 값은 %d 이고, 그떄의 i값은 %d 이다.", (min+570), savex); //i는 x[i]에서 몇번쨰 x인지 나타냄, +570을 한이유는 값이 음수로 나오는 걸 방지하기위해
        }

}