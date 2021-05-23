상관분석과 단순선형회귀분석
============================


#가짜 데이터 만들어 보기
가짜 데이터를 만들면서 파이썬을 복습해본다.   

클래스 선언하기   
--------------------

'''python  
class createDataFrame:   
      def __init__(self):   
        self.hairList = [[],[],[],[]] #데이터 프레임에 넣을 4차원 리스트   
        self.percentage = ([1,2,3,4,5]) #2~5=Y 흡연 or 음주 = 조금=2 /매우 많이=5   
                                        #1=N 흡연 or 음주 안한다.   
        self.tobacoo_per = 0 #피험자가 흡연할 확률   
        self.tobacooYn_per = 0 #흡연자가 탈모에 걸릴 확률   

        self.drink_per = 0 #피험자가 음주할 확률
        self.drinkYn_per = 0 #음주자가 탈모에 걸릴 확률

        self.tobacooAndDrink_per = 0 #흡연+음주자가 탈모에 걸릴 확률

        self.df = pd.DataFrame(columns=['피험자번호','흡연강도','음주강도','탈모강도'])
'''

클래스 안에 함수 만들기
-----------------------------
