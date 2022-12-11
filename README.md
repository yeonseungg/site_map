import csv, folium

count_csv = 0
area_list = []
base_map = folium.Map(location = [37.5642135, 127.0016985],zoom_start = 12)


#-----데이터 처리-----
find_file = input('파일 경로 or 이름을 입력하세요(파일명.csv) : ')
#route_file = "'.//"+find_file+"'"
#print(route_file)
csvfile = open(find_file, 'r', encoding='utf-8')
readcsv = csv.reader(csvfile)
for csv_inf in readcsv:
    if count_csv == 0:
        print('프로그램 실행 중...')
        count_csv += 1
    else:
        count_csv += 1
        latitude = float(csv_inf[9])
        longitude = float(csv_inf[10])
        marker_contents = (
            '<pre>'+csv_inf[1]+
            '<br>연면적(제곱미터) : '+csv_inf[2]+
            '<br>주용도 : '+csv_inf[4]+
            '<br>착공처리일 : '+csv_inf[5]+
            '<br>준공예정일 : '+csv_inf[6]+
            '</pre>')
        
        onearea_dict={}
        onearea_dict['지역이름'] = csv_inf[1]
        onearea_dict['연면적'] = csv_inf[2]
        onearea_dict['주용도'] = csv_inf[3]
        onearea_dict['착공처리일'] = csv_inf[4]
        onearea_dict['준공예정일'] = csv_inf[5]
        area_list.append(onearea_dict)
            
        folium.Marker([latitude, longitude], popup=marker_contents).add_to(base_map)
        #print(csv_inf)
        
#print(len(area_list))
csvfile.close()
base_map.save('./공사현황지도.html')

website_title = input('웹사이트에 들어갈 제목을 입력해주세요 : ')
    
#-----지도&웹사이트 만들기-----
base_map_read = open('./공사현황지도.html', 'r', encoding='utf-8')
make_website = open("website.html", 'w', encoding='utf-8')

#지도 html코드 가져와서 붙여넣기
lines = base_map_read.readlines()

for line in lines:
    if '<div class="folium-map"' in line:
        base_map_body = line

base_map_style = ''
for line in lines[27:34]:
    base_map_style += line

base_map_script = ''
for line in lines[44:-2]:
    base_map_script += line
    
areainf_list = '''<li>
                    <span class="지역이름"></span>
                    <br>연면적 : <span class="연면적"></span>
                    <br>주용도 : <span class="주용도"></span>
                    <br>착공처리일 : <span class="착공처리일"></span>
                    <br>준공예정일 : <span class="준공예정일"></span>
                </li>'''
areainf_list *= len(area_list)
nav_height = 140*len(area_list)

html_text = '''
<!DOCTYPE html>
<html>
<head>
   <meta charset="UTF-8">
    <meta http-equiv="content-type" content="text/html; charset=UTF-8" />
    
        <script>
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        </script>
    
    <style>html, body {width: 100%;height: 100%;margin: 0;padding: 0;}</style>
    <style>#map {position:absolute;top:0;bottom:0;right:0;left:0;}</style>
    <script src="https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js"></script>
    <script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js"></script>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css"/>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css"/>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css"/>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css"/>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css"/>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css"/>
    
    <meta name="viewport" content="width=device-width,
        initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
   <style>
        header{border-bottom:1px solid gray;padding:20px;}
        nav{border-right:1px solid gray; width:270px; height:'''+str(nav_height)+'''px; float:left;}
        article{padding-top:10px;width:800px;height:700px;position:fixed;float:left;transform:translate(280px);}
'''+base_map_style+'''
    </style>
</head>
<body>
    <header>
        <h1>'''+website_title+'''</h1>
    </header>
    <nav>
        <select id="search-select">
            <option value="지역이름">지역이름</option>
            <option value="연면적">연면적</option>
            <option value="주용도">주용도</option>
            <option value="착공처리일">착공처리일</option>
            <option value="준공예정일">준공예정일</option>
        </select>
        <input type="text" class="search-text" />
        <button id="btn">검색</button>
        <br>
        <br>

        <div>
            <br>=================================<br>
            검색 결과
            <br>=================================<br>
            <br><span id="result"></span>
        </div>
        <div>
            <br>=================================<br>
            지역 리스트
            <br>=================================<br>
            <br>
            <span id="areainf">
                '''+str(areainf_list)+'''
            </span>
        </div>
    </nav>
    <article>
'''+base_map_body+'''
    </article>
</body>
</html>
<script>
'''+base_map_script+'''
    let areainf = '''+str(area_list)+''';
    
    for (let i = 0; i < areainf.length; i++) {
        document.querySelectorAll("#areainf li .지역이름")[i].innerText = areainf[i].지역이름;
        document.querySelectorAll("#areainf li .연면적")[i].innerText = areainf[i].연면적;
        document.querySelectorAll("#areainf li .주용도")[i].innerText = areainf[i].주용도;
        document.querySelectorAll("#areainf li .착공처리일")[i].innerText = areainf[i].착공처리일;
        document.querySelectorAll("#areainf li .준공예정일")[i].innerText = areainf[i].준공예정일;
    }
    
    function searchFilter(data, type, search){
        return data.map((d) => {
            if (d[type].includes(search)) {
                return d;
            }
        });
    }
    
    function search() {
        let sel = document.getElementById("search-select").value;
        let text = document.getElementsByClassName("search-text")[0].value;
        
        let res = searchFilter(areainf, sel, text).filter((d) => d !== undefined);
        
        document.getElementById("result").innerText = res.map((d) => d.지역이름);
    }
    
    document.getElementById("btn").addEventListener("click", search);
</script>
'''
        

make_website.write(html_text)
base_map_read.close()
make_website.close()

print('프로그램이 완전히 종료되었습니다.')
fin = input('종료하려면 엔터키를 누르세요.')
