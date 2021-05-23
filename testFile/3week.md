상관분석과 단순선형회귀분석
============================


#가짜 데이터 만들어 보기
가짜 데이터를 만들면서 파이썬을 복습해본다.   
가짜 데이터의 전제 조건은 다음과 같다.
- 독립변수는 흡연강도와 음주강도이며 종속변수는 탈모증상이다.
- 흡연 강도와 음주 강도는 서로 상관이 있다. (상관분석)
- 흡연 또는 음주의 강도가 탈모 증상에 영향을 미친다. (회귀분석)
- 빈도수에 따른 5점 리커트 척도를 사용한다. 
- Q.나는 술, 음주를 1년동안    
- A.1=안했다. 2=조금 했다. 3=보통이다. 4=많이 했다. 5=매우 엄청 많이 했다.)
- 1000명을 대상으로 설문조사를 했다고 가정한다.
```python
#피험자가 흡연할 확률은 0.5
#피험자가 음주할 확률은 0.4
#흡연만하는 사람이 탈모에 걸릴 확률은 0.2
#음주만하는 사람이 탈모에 걸릴 확률은 0.4
#흡연-음주 하는사람이 탈모에 걸릴 확률은 0.7
#흡연 or 음주를 많이 할 수록 가중값이 10%씩 증가
```

가짜 데이터를 만드는 클래스 선언하기   
--------------------
```python  
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

        #빈 데이터 프레임 생성
        self.df = pd.DataFrame(columns=['피험자번호','흡연강도','음주강도','탈모강도'])
```

클래스 안에 가짜 데이터를 생성하는 함수 만들기
-----------------------------

### 1. 확률을 직접 핸들링 할 수 있도록 set 함수를 별도로 만든다.
```python
    def valuesSetting(self, tobacoo_per, tobacooYn_per,
                        drink_per, drinkYn_per,
                        tobacooAndDrink_per):
        self.tobacoo_per = tobacoo_per #피험자가 흡연자일 확률 
        self.tobacooYn_per = tobacooYn_per #흡연자가 탈모에 걸릴 확률


        self.drink_per = drink_per #피험자가 음주 확률
        self.drinkYn_per = drinkYn_per #음주자가 탈모에 걸릴 확률

        self.tobacooAndDrink_per = tobacooAndDrink_per #흡연+음주자가 탈모에 걸릴 확률
```

### 2. 확률에 가중값을 주는 함수를 만든다.
```python
    def Percentage(self,per,w):

        result = random.choices(self.percentage,weights=[per,1-per,w,w,w])
        return int(result[0])
```

### 3. 데이터 프레임을 만드는 함수를 만든다.
```python
    def create(self, subjectList):
        self.number(subjectList)
        self.Tobacco(self.tobacoo_per)
        self.Drinking(self.drink_per)
        self.Hair_lost()
```

### 4. 가상의 데이터셋을 만든다.


- 피험자의 식별번호를 생성한다. 안 만들어도 된다.
```python
#피험자 식별번호 생성
    def number(self, subjectList):
        numberList=[]
        zero = '0' #자릿수
        max = 6 #맥스 자릿수

        for num in subjectList:
            text = ''

            for i in range(0, max-len(str(num)), 1):
                text += zero

            text = text+str(num)
            numberList.append(text)

        self.hairList[0].extend(numberList)
```

- 흡연 강도를 설정한다.  
```python
    #흡연 강도 생성
    def Tobacco(self, w) :
        numberList=[]
        for i in range(0, len(self.hairList[0]), 1):
            if(self.Percentage(self.tobacoo_per, w) == 2): #확률에 가중값을 주는 함수에서 50%확률로 흡연자인지 아닌지를 설정한 후
                numberList.append(random.randint(2,5)) #흡연자일 경우 랜덤으로 2~5의 값을 준다.
            else:
                numberList.append(1)
        self.hairList[1].extend(numberList)
```


- 음주 강도를 설정한다.
```python
 #음주 강도 생성
    def Drinking(self, w):
        numberList=[]
        for i in range(0, len(self.hairList[0]), 1):
            if (self.Percentage(self.drink_per, w) == 2): #확률에 가중값을 주는 함수에서 40%확률로 음주자인지 아닌지를 설정한 후
                numberList.append(random.randint(2, 5)) #음주자일 경우 랜덤으로 2~5의 값을 준다.
            else:
                numberList.append(1)
        self.hairList[2].extend(numberList)
```


- 탈모강도를 흡연, 음주 강도에 따라 다르게 설정되게끔 한다.
```python
#탈모강도 생성
    def Hair_lost(self):
        #데이터 프레임에 데이터 추가
        self.df.loc[:,'피험자번호'] = self.hairList[0]
        self.df.loc[:,'흡연강도'] = self.hairList[1]
        self.df.loc[:,'음주강도'] = self.hairList[2]
        numberList=[]
        for index, data in self.df.iterrows(): #한 행의 데이터를 data 변수에 집어넣는다.
            if data[1]==1 and data[2]==1: #흡연+음주 X
                numberList.append(1)
            elif data[1]!=1 and data[2]!=1: #흡연+음주 O
                val = data[1]+data[2]
                val = val / 10
                val = val / 2
                w = val + self.tobacooAndDrink_per
                numberList.append(self.Percentage(self.tobacooAndDrink_per, w))
            elif data[1]!=1 and data[2]==1: #흡연O 음주X
                w = self.tobacooYn_per+(data[1] / 10)
                numberList.append(self.Percentage(self.tobacooYn_per, w))
            elif data[1]==1 and data[2] !=1: #흡연X 음주O
                w = self.drinkYn_per+(data[2]/10)
                numberList.append(self.Percentage(self.drinkYn_per ,w))
        self.hairList[3].extend(numberList)
        self.df.loc[:,'탈모강도'] = self.hairList[3]

```


- 리턴 값 만들기
```python
    def getDataFrame(self):
        return self.df
```


가짜데이터 생성하기
-------------------------
```python
if __name__ == '__main__':
      
    margi = createDataFrame()

    tobacoo_per = 0.5  # 피험자가 흡연할 확률 (독립1)
    tobacooYn_per = 0.2  # 흡연자가 탈모에 걸릴 확률 (독립1-종속)

    drink_per = 0.4  # 피험자가 음주할 확률 (독립2)
    drinkYn_per = 0.3  # 음주자가 탈모에 걸릴 확률 (독립2-종속)

    tobacooAndDrink_per = 0.5  # 흡연+음주자가 탈모에 걸릴 확률 (종속)

    #독립1, 독립1-종속,
    #독립2, 독립2-종속,
    #(독립1+독립2)-종속
    margi.valuesSetting(tobacoo_per, tobacooYn_per,
                        drink_per, drinkYn_per,
                        tobacooAndDrink_per)

    margi.create(list(range(1,1001)))
    print(margi.getDataFrame())
"""
+-----+--------------+------------+------------+------------+   
|     |   피험자번호 |   흡연강도 |   음주강도 |   탈모강도 |   
|-----+--------------+------------+------------+------------|   
|   0 |       000001 |          4 |          1 |          2 |   
|   1 |       000002 |          2 |          1 |          2 |   
|   2 |       000003 |          1 |          1 |          1 |   
|   3 |       000004 |          1 |          1 |          1 |   
|   4 |       000005 |          1 |          5 |          1 |   
|   5 |       000006 |          1 |          1 |          1 |   
|   6 |       000007 |          1 |          1 |          1 |   
|   7 |       000008 |          2 |          1 |          2 |   
|   8 |       000009 |          4 |          1 |          3 |   
|   9 |       000010 |          1 |          1 |          1 |   
|  10 |       000011 |          1 |          1 |          1 |   
|  11 |       000012 |          1 |          2 |          4 |   
|  12 |       000013 |          1 |          4 |          2 |   
|  13 |       000014 |          1 |          1 |          1 |   
|  14 |       000015 |          1 |          1 |          1 |   
|  15 |       000016 |          1 |          4 |          4 | 
"""
```
