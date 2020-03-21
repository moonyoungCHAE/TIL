### map

#### 순회
```
	for (auto it = m.begin(); it != m.end(); it++){
		cout << "key : " << it->first << " " << "value : " << it->second << '\n';
	}
```

#### sort
1. vector를 활용하여 정렬
(1) map을 vector에 넣는다.
(2) sort한다

``` C++
// map을 vector에 추가
for (it2=mymap.begin(); it2!=mymap.end(); it2++) 
  {
    vec.push_back(make_pair(it2->first, it2->second));
  }
  
// sort
#include <algorithm> // for sort function

bool sortByVal(const pair<string, int> &a, 
               const pair<string, int> &b) { 
    return (a.second < b.second); 
} 
  sort(vec.begin(), vec.end(), sortByVal); 
  
// 접근
vec[i]->first
```