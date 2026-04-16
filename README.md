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


    import json # JSON 파일을 다루기 위해 맨 위에 추가해 주세요.
    import random
    import time # 시간 측정을 위한 모듈 (파일 맨 위에 추가)
    import glob # 파일 목록을 찾기 위해 맨 위에 추가해 주세요.
    import os   # 파일 경로를 다루기 위해 맨 위에 추가해 주세요.

    #1단계: MAC 연산 함수와 부동소수점 비교 로직 만들기 (핵심 엔진)
    
    def calculate_mac(pattern, filter_matrix):
        """
        패턴과 필터의 MAC 연산을 수행합니다.
        """
        # 패턴의 행(row)과 열(col)의 크기를 구합니다.
        n = len(pattern)    #이거는 pattern의 행 개수만 구하는 용도. 지금은 행과 열의 크기가 동일해서 이렇게만 해도 됨
    
    # # 만약에 행과 열의 개수가 다르면 어떻게 해야할까?  (기본 전제는 모든 행의 길이는 같다. 그렇지 않으면 오류가 납니다!)
    # #   pattern = [
    #         [1, 2, 3],
    #         [4, 5, 6]
    #     ]
    #  -> rows = len(patter) --- 행 개수는 동일하게 함. 2가 나옴.
    #  -> cols = len(pattern[0]) --- 1행의 길이를 구한다. 즉 열의 개수를 구함. 3이 나옴.
        
        total_score = 0.0 # 결과를 저장할 변수
    
        # 이중 for문을 사용해 2차원 리스트의 모든 원소에 접근합니다.
        for i in range(n):
            for j in range(n):
                # 같은 위치(i, j)의 값을 곱해서 더합니다.
                total_score += pattern[i][j] * filter_matrix[i][j]
    
        return total_score
    
    def compare_scores(score_cross, score_x):
        """
        두 필터의 점수를 비교하여 판정 결과를 반환합니다.
        """
        epsilon = 1e-9 # 0.000000001 (매우 작은 수)
    
        # 두 점수의 차이의 절댓값이 epsilon보다 작으면 동점으로 판정
        if abs(score_cross - score_x) < epsilon:
            return "UNDECIDED"
        elif score_cross > score_x:
            return "Cross"
        else:
            return "X"
        
    
    # 2단계: 3x3 콘솔 입력 기능 만들기
    
    def input_3x3_matrix(matrix_name):
        """
        사용자로부터 3x3 행렬을 입력받아 반환하는 함수입니다.
        """
        matrix = []    #입력하기 위한 빈 리스트를 하나 생성. 나중에는 입력으로 인해 행과 열이 채워짐.
        print("3x3 행렬을 입력하세요 (각 행마다 3개의 숫자를 공백으로 구분하여 입력):")
        print("각 줄에 3개의 숫자를 공백으로 구분해 입력하세요. (총 3줄)")
    
        while True: # 올바르게 입력할 때까지 무한 반복하겠다.
            matrix = []  # 반복해서 시작할 때, matrix는 다시 빈 리스트가 된다. 즉, 새로 시작하니 예전 내용은 버림.
            try:
                for i in range(3):
                    # 1. 한 줄 입력받기    ex. 1 0 1
                    row_input = input(f"{i+1}번째 줄: ")
    
                    # 2. 공백 기준으로 자르고, 숫자로 변환하여 리스트로 만들기    ---> row_input에 입력한 문자열을 숫자 리스트로 바꾸기
                    # 예: "1 0 1" -> [1.0, 0.0, 1.0]
                    row = [float(x) for x in row_input.split()]
                    #[row_input.split()] --> row_input = "1 0 1" 이면 .spilit이 공백을 기준으로 나눠서 "1", "0", "1"로 바꿈. -> ["1", "0", "1"]이 됨.
         #[float(x) for x in row_input.split()] ->  row_input.split()으로 만들어진 list에서 index 순서대로 x에 하나씩 부여한 후, float(x)로 x를 숫자로 변환.
                    #지금 리스트에는 "1"로 문자열로 들어가있으니, float()로 숫자로 바꿔주는 것 입니다.
            
    
                    # 3. 열의 개수가 3개인지 확인
                    if len(row) != 3:     #현재 row의 자료형은 list. len(row)는 row라는 list 안에 몇개가 들어있냐를 물어봄.
                        raise ValueError("각 행에는 정확히 3개의 숫자가 필요합니다.")
                    
                    # 변환된 한 줄(row)을 전체 행력(matrix)에 추가합니다!  --> 제대로 입력된 한 줄을 전체 행렬에 추가한다.
                    matrix.append(row)   #처음에 만든 빈 list가 matix. 여기에 append(row)로 matix라는 빈 리스트에 row에 있는 list를 추가함. 
                    
    
                # 에러 없이 여기가지 왔다면 올바른 2차원 리스트가 완성된 것!
                return matrix
            #맨 처음에 try하고 for i in range(3)로 총 3줄을 입력받기로 함. 3줄이 정상적으로 입력되면, 완성된 matrix를 반환한다.   
        
            
            except ValueError:
                 # 숫자가 아닌 문자를 넣었거나, 개수가 틀렸을 때 실행됨
                print("❌ 입력 형식 오류: 각 줄에 3개의 숫자를 공백으로 구분해 입력하세요. 처음부터 다시 입력합니다.\n")
            # ValueError -- float("a")같은 반환 실패, raise ValueError: - 직접 ValueError를 발생시킨다.
            # 아까 위에서 if ~~ ValueError(~~)를 했음. 이건 row가 3개가 아니면 이것을 ValueError라 생각한다는 것 입니다!
    
    
    # 3단계: 라벨 정규화 및 JSON 파일 읽기
    
    # 1. 라벨 정규화 함수
    def normalize_label(label):
        """
        다양한 형태의 라벨을 표준 형태('Cross' 또는 'X')로 변환합니다.
        """
        # 소문자로 바꾸고 양옆 공백을 제거하여 비교하기 쉽게 만듭니다.
        label = label.lower().strip() #lower()로 문자열을 다 소문자로, strip()으로 공백 제거   
        #Ex. label="  CRoSS " -> label.lower().strip() = "cross"
        
        if label in ['+', 'cross']:
            return 'Cross'    #label이 ['+', 'cross']에 있으면 'Cross'를 반환함. -> 이름을 Cross로 통일한다.
        elif label == 'x':
            return 'X'
        else:
            return label # 예상치 못한 라벨은 일단 그대로 반환. +, cross가 아니고 x도 아니면 그냥 label에 입력된 그대로를 반환함.
    
    # 2. JSON 파일 읽기 및 검증 함수
    def load_json_data(file_path):
        """
        JSON 파일을 읽고, 3x3 크기인지 검증한 뒤 행렬과 정규화된 라벨을 반환합니다.
        """
        try:
            # 파일 열기
            with open(file_path, 'r', encoding='utf-8') as f:  
                data = json.load(f)
                
            matrix = data.get('matrix')
            label = data.get('label')
            
            # [검증] 행렬이 존재하는지, 행이 3개인지, 모든 열이 3개인지 확인
            if not matrix or len(matrix) != 3 or any(len(row) != 3 for row in matrix):
                print(f"FAIL: {file_path} 파일의 행렬 크기가 3x3이 아닙니다.")
                return None, None # 프로그램 종료 대신 None을 반환하여 안전하게 처리
                
            # 라벨 정규화 적용
            normalized_label = normalize_label(label)
            
            return matrix, normalized_label
            
        except FileNotFoundError:
            print(f"FAIL: {file_path} 파일을 찾을 수 없습니다.")
            return None, None
        except json.JSONDecodeError:
            print(f"FAIL: {file_path} 파일이 올바른 JSON 형식이 아닙니다.")
            return None, None
    
    
    # 4단계: 여러 JSON 파일 자동 채점기 만들기 (모드 2)
    
    def run_json_test_mode(folder_path):
        """
        지정된 폴더 내의 모든 JSON 파일을 읽어 MAC 연산을 수행하고,
        정답과 비교하여 정확도를 계산합니다.
        """
        print(f"\n=== JSON 테스트 모드 시작 (대상 폴더: {folder_path}) ===")
    
        # 1. 기준이 되는 필터 A(Cross)와 필터 B(X)를 미리 정의해 둡니다.
        # (매번 입력받을 수 없으니 고정해 둡니다)
        filter_cross = [
            [0, 1, 0], 
            [1, 1, 1], 
            [0, 1, 0]
        ]
        filter_x = [
            [1, 0, 1], 
            [0, 1, 0], 
            [1, 0, 1]
        ]
    
        # 2. 폴더 안의 모든 .json 파일 찾기
        file_pattern = os.path.join(folder_path, '*.json')
        file_list = glob.glob(file_pattern)
    
        if not file_list:
            print("❌ 테스트할 JSON 파일이 없습니다. 폴더 경로를 확인해 주세요.")
            return
    
        total_files = len(file_list)
        pass_count = 0
    
        # 3. 찾은 파일들을 하나씩 꺼내서 검사하기
        for file_path in file_list:
            # 우리가 3단계에서 만든 함수로 데이터 읽기!
            pattern, true_label = load_json_data(file_path)
    
            if pattern is None:
                continue # 파일 읽기에 실패하면 다음 파일로 넘어감
    
            # 우리가 1단계에서 만든 함수로 MAC 점수 계산 및 판정!
            score_cross = calculate_mac(pattern, filter_cross)
            score_x = calculate_mac(pattern, filter_x)
            prediction = compare_scores(score_cross, score_x)
    
            # 4. 예측 결과와 실제 정답(true_label) 비교
            if prediction == true_label:
                result_text = "PASS"
                pass_count += 1
            else:
                result_text = "FAIL"
    
            # 파일 이름만 깔끔하게 뽑아서 결과 출력
            file_name = os.path.basename(file_path)
            print(f"[{result_text}] {file_name} | 예측: {prediction} | 정답: {true_label}")
    
        # 5. 최종 정확도 계산 및 출력
        accuracy = (pass_count / total_files) * 100
        print("\n===================================")
        print(f"총 테스트 파일: {total_files}개")
        print(f"통과(PASS): {pass_count}개")
        print(f"정확도(Accuracy): {accuracy:.2f}%")
        print("===================================")
    
    # 5단계: 크기별 성능 분석 (연산 시간 측정)    
    
    def measure_performance():
        """
        5x5, 13x13, 25x25 크기의 행렬에 대해 MAC 연산 시간을 측정합니다.
        """
        print("\n=== 5단계: 행렬 크기별 MAC 연산 시간 측정 ===")
        
        # 측정할 행렬의 크기들 (N x N)
        sizes = [5, 13, 25]
        iterations = 10 # 10회 반복 측정
        
        for n in sizes:
            # 1. 테스트를 위한 N x N 크기의 임의의 패턴과 필터 생성 (0과 1로 채움)
            pattern = [[random.choice([0, 1]) for _ in range(n)] for _ in range(n)]
            filter_matrix = [[random.choice([0, 1]) for _ in range(n)] for _ in range(n)]
            
            total_time = 0.0
            
            # 2. 10회 반복하며 시간 측정
            for _ in range(iterations):
                # --- 측정 시작 ---
                start_time = time.perf_counter() # 아주 정밀한 타이머 시작
                
                calculate_mac(pattern, filter_matrix) # 순수 연산만 실행!
                
                end_time = time.perf_counter()   # 타이머 종료
                # --- 측정 종료 ---
                
                total_time += (end_time - start_time)
                
            # 3. 평균 시간 계산 및 출력
            avg_time = total_time / iterations
            print(f"{n:>2} x {n:>2} 행렬 평균 연산 시간: {avg_time:.8f} 초")
    
    if __name__ == "__main__":
        # 기존 모드 1(수동 입력) 코드는 모두 지우거나 주석 처리합니다.
    
        # ---------------------------------------------------------
        # [1부] JSON 자동 채점 시스템
        # ---------------------------------------------------------
        print("\n=== [1부] JSON 자동 채점 시스템 ===")
        
        # ⚠️ 4단계에서 만드신 JSON 채점 함수를 여기서 호출합니다.
        # (함수 이름이 다르다면 본인의 함수 이름으로 바꿔주세요!)
        run_json_test_mode("./test_data") 
        
        print("\n") # 1부와 2부 사이를 깔끔하게 띄워줍니다.
    
        # ---------------------------------------------------------
        # [2부] 행렬 크기별 MAC 연산 시간 측정
        # ---------------------------------------------------------
        print("=== [2부] 행렬 크기별 MAC 연산 시간 측정 ===")
        
        # 5단계에서 만든 성능 측정 함수를 호출합니다.
        measure_performance()




<img width="1068" height="1169" alt="Screenshot 2026-04-16 at 8 56 38 PM" src="https://github.com/user-attachments/assets/63f70656-77f0-401d-8b98-48ba44dd13c1" />

<img width="1059" height="1140" alt="Screenshot 2026-04-16 at 8 56 54 PM" src="https://github.com/user-attachments/assets/dc9643cb-1b22-48c9-ab6a-caf8f6af28de" />

<img width="920" height="1154" alt="Screenshot 2026-04-16 at 8 57 09 PM" src="https://github.com/user-attachments/assets/d991cb54-7ebf-4664-9a5b-cc146bd18673" />

<img width="906" height="1157" alt="Screenshot 2026-04-16 at 8 57 26 PM" src="https://github.com/user-attachments/assets/27c73a3a-ad47-4538-87a7-c5c7a51cb8cc" />

<img width="645" height="226" alt="Screenshot 2026-04-16 at 8 57 31 PM" src="https://github.com/user-attachments/assets/16df3ff5-1bad-4975-ab5d-7cc0aec16b0a" />

















