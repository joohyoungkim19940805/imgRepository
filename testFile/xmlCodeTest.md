XML 노드 가져오기 / DataFream에 저장하기 작업에 다형성을 추가하여 작업 효율 높이기
============================

대상이 되는 XML 파일
----------------
- 한국금융감독원에 등록된 기업체의 고유번호, 업체이름, 주식상장코드, 업데이트날짜로 이루어져있다.   
- 전자공시 opendart restapi를 이용하기 위해서는 기업체의 고유번호를 key값으로 이용해야 하기 때문에 해당 xml 데이터가 필요하다.   
- 단 작성자는 주식시장에 상장된 기업체만을 대상으로 할 예정이기 때문에 <stock_code>가 빈값인 기업체의 데이터는 필요 없다.   
> (ex : 다코)
```xml
<result>
    <list>
        <corp_code>00434003</corp_code>
        <corp_name>다코</corp_name>
        <stock_code> </stock_code>
        <modify_date>20170630</modify_date>
    </list>
    <list>
        <corp_code>00434456</corp_code>
        <corp_name>일산약품</corp_name>
        <stock_code> </stock_code>
        <modify_date>20170630</modify_date>
    </list>
    <list>
        <corp_code>00126380</corp_code>
        <corp_name>삼성전자</corp_name>
        <stock_code>005930</stock_code>
        <modify_date>20201209</modify_date>
    </list>
</result>
```

기존 코드의 문제점
---------------------------
- 파일 경로를 반드시 알아야 파싱할 수 있었다.
> ex : (C:xml\\XMLFILE.xml)
- 노드의 tag 이름을 반드시 알아야 파싱할 수 있었다.
> ex : (<list> <corp_code>, <corp_name>, <stock_code>, <modify_date> </list>)
- 데이터 프레임에서 {'컬럼 이름' : 데이터}를 지정해줄 때 '컬럼 이름'을 무조건 하드코딩으로 해야 해서 손가락이 아팠다.
```python
if __name__ == '__main__':
    tree = ET.parse('xmlDir/CORPCODE.xml')
    root = tree.getroot()
    dataList = [[],[],[],[]]
    for content in root.iter('list'):
        if content.findtext('stock_code') != ' ' and len(content.findtext('stock_code')) > 1:
            dataList[0].append(content.findtext('corp_code'))
            dataList[1].append(content.findtext('corp_name'))
            dataList[2].append(content.findtext('stock_code'))
            dataList[3].append(content.findtext('modify_date'))

    df = pd.DataFrame(
        {
        'corp_code' : dataList[0],
        'corp_name' : dataList[1],
        'stock_code' : dataList[2],
        'modify_date' : dataList[3]
        })
    print_df(df)
```

해결방안
----------------------------------
- 1. 파일 이름만으로 파일경로까지 찾아서 해당 파일을 매핑할 수 있어야 한다.
- 2. 노드의 이름을 알지 못해도 xml 데이터를 파싱할 수 있어야 한다.
- 3. 데이터 프레임을 만들 때 별도의 하드코딩을 안 해도 된다.
 
0.import info
```python
import xml.etree.ElementTree as et # XML 파싱을 위한 패키지
import os as os                    # 파일 이름으로 파일 경로 찾기를 위한 os 패키지
import pandas as pd                # 판다스
import time as time                # 함수 실행 속도 비교를 위한 time 패키지
from tabulate import tabulate      # 데이터프레임을 예쁘게 출력할 때 쓸 패키지
#from multiprocessing import Pool  # 멀티프로세스로 특정 함수를 자바 쓰레드처럼 돌릴 때 쓰는 패키지 쓰레드와는 개념이 다르다. 쓰려다가 말았다.
#import sys, os                    # 에러로그 해결책 중 하나, 하지만 안 쓴다.
    
def print_df(data):                #데이터 프레임을 예쁘게 출력해주는 함수
print(tabulate(data, headers='keys', tablefmt='psql'))


class create: #사용하게 될 클래스
    def __init__(self):
        self.dir_root=''
```
- 함수 목록
> create_element_tree()    : 최상위에 있는 노드를 가져오는 함수 /

> find_file()              : 파일 이름으로 해당 파일이 있는 경로까지 매핑해줄 함수 /

> find_lowest_tag()        : 최상위 노드에서 최하위 노드까지 내려가면서 최하위 노드와 그 부모 노드를 같이 리턴해주는 함수 

> check_validation()       : 최하위 노드와 그 부모 노드를 매개변수로 받으면 최하위 노드의 갯수만큼 차원 배열을 가지는 빈 리스트와 최하위 노드들의 이름을 리스트로 set해주고, 최하위 노드의 갯수를 리턴해주는 함수 / 추후에 기능을 분리할 예정

> create_stock_dataFrame() : 최하위 노드와 그 부모 노드를 매개변수로 받으면 최하위 노드의 갯수만큼 차원을 가지는 빈 리스트에 데이터를 집어넣고, 노드에 알맞는 데이터 프레임을 생성시켜주는 함수 / 추후에 기능을 분리할 예정   
    
> create_list() : 숫자를 매개변수로 받으면 그 숫자만큼 차원을 가지는 빈 리스트를 생성하는 함수
    
> getColumn_name_list() : 컬럼 이름을 리스트로 리턴해주는 함수, check_validation()을 먼저 호출해야 사용 가능한 함수
    
> getStock_list() : xml 데이터를 리스트 형태로 리턴해주는 함수

1. 파일 이름만으로 파일경로까지 찾아서 해당 파일을 매핑하기
-------------------------------------------
