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
