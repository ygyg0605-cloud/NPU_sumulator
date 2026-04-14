# MAC 기반 패턴 판별 콘솔 애플리케이션

## 1. 프로젝트 개요

본 프로젝트는 MAC(Multiply-Accumulate) 연산을 이용하여 입력 패턴이
Cross인지 X인지 판별하는 Python 콘솔 애플리케이션이다.

## 2. 주요 기능

-   MAC 연산 기반 패턴 판별
-   JSON 자동 채점 시스템
-   라벨 정규화 (Cross / X)
-   epsilon 기반 부동소수점 비교
-   성능 분석 (O(N²))

## 3. 실행 방법

python main.py

## 4. 핵심 개념

MAC = Σ(pattern × filter) 시간 복잡도 = O(N²)

## 5. 결과 리포트

-   데이터/스키마 문제
-   라벨 문제
-   수치 비교 문제

epsilon 정책과 정규화를 통해 안정적인 결과를 얻는다.
