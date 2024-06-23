import myFilters        # 날짜: 2024/06/23 16:58, 작성자:신재우 용도:필터가 정의된 클래스 불러오기
import myMeasures       # 날짜: 2024/06/23 16:58, 작성자:신재우 용도:데이터의 측정을 대신해줄 함수 또는 클래스 불러오기
import matplotlib.pyplot as plt    # 날짜: 2024/06/23 16:58, 작성자:신재우  용도: 그래프 출력을 위한 모듈 불러오기

class LowPassFilter:
    # 날짜: 2024/06/23 17:00, 작성자: 신재우
    # 용도: LowPassFilter,새로운 클래스 정의
    def __init__(self, numOfData, prevAvg=0):
        # 날짜: 2024/06/23 17:02, 작성자: 신재우
        # 용도: LowPassFilter 클래스의 생성자
        self.xBuf = []  # 날짜: 2024/06/23 17:02, 작성자: 신재우 용도:버퍼를 초기화한다.
        self.firstRun = 1  # 날짜: 2024/06/23 17:02, 작성자: 신재우 용도:처음 실행되는지 여부를 나타내는 플래그 (1은 True를 의미).
        self.index = 0 # 날짜: 2024/06/23 17:02, 작성자: 신재우 용도:버퍼의 현재 인덱스를 초기화한다.
        self.numOfData = numOfData # 날짜: 2024/06/23 17:02, 작성자: 신재우 용도:버퍼의 최대 크기를 설정한다.
        self.prevAvg = prevAvg # 날짜: 2024/06/23 17:02, 작성자: 신재우 용도:이전 평균값을 설정한다.
        if self.prevAvg != 0: # 날짜: 2024/06/23 17:02, 작성자: 신재우 용도: 이전의 평균값을 주는 경우 버퍼에 채움
            for _ in range(self.numOfData):
                self.xBuf.append(self.prevAvg)
            self.index = 0   # 날짜: 2024/06/23 17:02, 작성자: 신재우 용도: 인덱스를 초기화한다.
            self.firstRun = 0   # 날짜: 2024/06/23 17:02, 작성자: 신재우 용도 : 처음 실행되지 않음을 나타내는 플래그를 설정한다(0은 False를 의미).

    def lowpassFilter(self, xn):
        """
         날짜: 2024/06/23 17:03, 작성자: 신재우
         용도: 새로운 입력 xn에 대해 저주파 필터를 적용하고 평균값을 반환

         입력:
          - xn: 현재의 입력 데이터 포인트

         반환:
         - avg: 저주파 필터링된 평균값
         """
        alpha = 0.97   # 날짜: 2024/06/23 17:03, 작성자: 신재우 용도: 저주파 필터의 상수 alpha 설정
        if self.firstRun == 1:  # 날짜: 2024/06/23 17:04, 작성자: 신재우 용도: 초기에 평균값을 주지 않고 측정값이 바로 들어오는 경우 모든 버퍼를 채움
            for _ in range(self.numOfData):  # 날짜: 2024/06/23 17:04, 작성자: 신재우 용도 : 버퍼에 초기 데이터를 입력한다.
                self.xBuf.append(xn)
            avg = xn  # 날짜: 2024/06/23 17:04, 작성자: 신재우 용도: 초기 상태에서의 평균값을 설정.
            self.firstRun = 0  # 날짜: 2024/06/23 17:04, 작성자: 신재우 용도:초기 실행 여부를 업데이트한다.
            self.index = 1   # 날짜: 2024/06/23 17:04, 작성자: 신재우 용도: 버퍼의 시작 인덱스를 설정.
        else:
            # 날짜: 2024/06/23 17:05, 작성자: 신재우 용도 : 초기 실행이 아닌 경우 저주파를 계산하고 새로운 값을 버퍼에 저장한다.
            avg = alpha * self.prevAvg + (1 - alpha) * xn
            # 날짜: 2024/06/23 17:05, 작성자: 신재우 용도: 새로운 값을 버퍼에 저장
            self.xBuf[self.index] = xn
            # 인덱스를 조정.
            self.index += 1 # 날짜: 2024/06/23 17:05, 작성자: 신재우 용도:다음 인덱스로 이동한다.
            self.index %= self.numOfData   # 날짜: 2024/06/23 17:06, 작성자: 신재우 용도:원형 버퍼 스타일로 인덱스를 조정.

        self.prevAvg = avg  # 날짜: 2024/06/23 17:06, 작성자: 신재우 용도:이전 평균값을 현재 계산된 평균값으로 업데이트한다.

        return avg    # 날짜: 2024/06/23 17:06, 작성자: 신재우 용도:저주파 필터링된 평균값을 반환한다.

# 측정 간격과 평균 계산에 사용할 시간 간격을 설정한다.
samplingInterval = 0.01  # 날짜: 2024/06/23 17:10, 작성자: 신재우, 용도: 측정 간격, 0.01초
timeAvg = 0.5            # 날짜: 2024/06/23 17:10, 작성자: 신재우, 용도: 평균 계산에 사용할 시간 간격, 0.5초
nSamples = int(timeAvg / samplingInterval)  # 날짜: 2024/06/23 17:11, 작성자: 신재우, 용도: 평균을 내는 데이터의 개수

fileName = "sonarData.txt"  # 날짜: 2024/06# /23 17:12, 작성자: 신재우, 용도: 데이터가 저장된 파일의 이름을 설정한다.

# 날짜: 2024/06/23 17:13, 작성자: 신재우, 용도 데이터 저장을 위한 리스트 변수를 초기화한다
dataMeasured = [[], []]  # 날짜: 2024/06/23 17:13, 작성자: 신재우, 용도: 측정값이 저장되는 파일 이름
dataAverage1 = [[], []]   # 날짜: 2024/06/23 17:13, 작성자: 신재우, 용도:lowpassfilter의 필터링된 데이터 저장.
dataAverage2 = [[], []]   # 날짜: 2024/06/23 17:13, 작성자: 신재우, 용도: 이동 평균 필터링된 데이터 저장,lowpassfilter와 비교하기 위해 사용

# 객체를 생성한다.
objSonarData = myMeasures.clsGetDataFromFile(fileName)  # 날짜: 2024/06/23 17:14, 작성자: 신재우, 용도: 파일로부터 데이터를 가져올 객체 생성
lowpassFilter = LowPassFilter(numOfData=nSamples)  # 날짜: 2024/06/23 17:14, 작성자: 신재우, 용도: 저주파 필터 객체 생성
movAvgFilter = myFilters.clsMovAvgFilter(nSamples)  # 날짜: 2024/06/23 17:14, 작성자: 신재우, 용도: 이동 평균 필터 객체 생성

# 데이터를 처리하고 필터링한다.
for i in range(objSonarData.numOfData):
    xn = objSonarData.GetDataFromFile()  # 날짜: 2024/06/23 17:15, 작성자: 신재우, 용도: 파일로부터 데이터를 가져옴
    avg1 = lowpassFilter.lowpassFilter(xn)  # 날짜: 2024/06/23 17:15, 작성자: 신재우, 용도: 저주파 필터를 적용하여 평균값 계산
    avg2 = movAvgFilter.MovAvgFilter(xn)  # 날짜: 2024/06/23 17:15, 작성자: 신재우, 용도: 이동 평균 필터를 적용하여 평균값 계산
    sec = int(i * samplingInterval * 100) / 100  # 날짜: 2024/06/23 17:15, 작성자: 신재우, 용도: 시간 계산, 자리수 오류 제거
    dataMeasured[0].append(sec)  # 날짜: 2024/06/23 17:16, 작성자: 신재우, 용도: 측정 시간 저장
    dataMeasured[1].append(xn)  # 날짜: 2024/06/23 17:16, 작성자: 신재우, 용도: 측정 데이터 저장
    dataAverage1[0].append(sec)  # 날짜: 2024/06/23 17:16, 작성자: 신재우, 용도: 저주파 필터링된 시간 저장
    dataAverage1[1].append(avg1)  # 날짜: 2024/06/23 17:16, 작성자: 신재우, 용도: 저주파 필터링된 데이터 저장
    dataAverage2[0].append(sec)  # 날짜: 2024/06/23 17:16, 작성자: 신재우, 용도: 이동 평균 필터링된 시간 저장
    dataAverage2[1].append(avg2)  # 날짜: 2024/06/23 17:16, 작성자: 신재우, 용도: 이동 평균 필터링된 데이터 저장

# 처리한 결과를 그래프로 출력한다.
plt.plot(dataMeasured[0], dataMeasured[1], 'b')  # 날짜: 2024/06/23 17:17, 작성자: 신재우, 용도: 원본 데이터를 파란색으로 플로팅
plt.plot(dataAverage1[0], dataAverage1[1], 'r')  # 날짜: 2024/06/23 17:17, 작성자: 신재우, 용도: 저주파 필터링된 데이터를 빨간색으로 플로팅
plt.plot(dataAverage2[0], dataAverage2[1], 'g')  # 날짜: 2024/06/23 17:17, 작성자: 신재우, 용도: 이동 평균 필터링된 데이터를 녹색으로 플로팅
plt.plot(dataMeasured[0], dataMeasured[1], 'bs')  # 날짜: 2024/06/23 17:17, 작성자: 신재우, 용도: 원본 데이터 포인트를 파란색 점으로 표시
plt.plot(dataAverage1[0], dataAverage1[1], 'ro')  # 날짜: 2024/06/23 17:17, 작성자: 신재우, 용도: 저주파 필터링된 데이터 포인트를 빨간색 점으로 표시
plt.plot(dataAverage2[0], dataAverage2[1], 'go')  # 날짜: 2024/06/23 17:17, 작성자: 신재우, 용도: 이동 평균 필터링된 데이터 포인트를 녹색 점으로 표시
plt.xlabel("Time (sec)")  # 날짜: 2024/06/23 17:17, 작성자: 신재우, 용도: x축 레이블 설정
plt.ylabel("Height (m)")  # 날짜: 2024/06/23 17:18, 작성자: 신재우, 용도: y축 레이블 설정
plt.title("Experiment Result")  # 날짜: 2024/06/23 17:18, 작성자: 신재우, 용도: 그래프 제목 설정
plt.legend(["Measure", "Low Pass Filter", "Moving Average Filter"])  # 날짜: 2024/06/23 17:18, 작성자: 신재우, 용도: 범례 설정
plt.grid()  # 날짜: 2024/06/23 17:20, 작성자: 신재우, 용도: 그리드 설정
plt.show()  # 날짜: 2024/06/23 17:20, 작성자: 신재우, 용도: 그래프를 보여준다.
