# New Korean Word Corpus

한국어 분석을 위하여 만들어진 신조어 코퍼스 입니다.
약 9천만건의 NAVER 뉴스를 크롤링하고, 그 중 뉴스 헤드라인에서 모든 명사를 추출하였습니다.
명사 추출은 비지도학습 기반의 Noun Extractor을 제공하는 `soynlp` 패키지로 진행되었으며, 신조어 판별을 위한 baseline으로 세종 코퍼스 (Sejong Corpus) 명사들을 사용하였습니다. 데이터는 원 단어장 JSON 및 형태소 분석기에 빌드 가능한 형태 등 총 2가지로 제공합니다.

## Spec
### Data

NAVER 뉴스에서 제공하는 총 8가지 카테고리 (정치, 경제, 사회, 문화, 스포츠, 연예, 세계, IT/과학) 에서 2002년 05월 06일부터 2019년 09월 29일까지의 뉴스 헤드라인을 수집하였습니다. (총 9,800 만여 건)

Baseline으로 사용된 세종 코퍼스는 `Mecab` 형태소 분석기 내의 기본 사전을 활용하였습니다. 일반명사 (NNG)와 고유명사 (NNP)를 합하여 총 707,486 개가 사용되었습니다.

### Process

먼저, 수집된 뉴스 헤드라인에서 명사를 모두 추출하였습니다. 카테고리 별로 `doublelinecorpus` 문서를 만들어 `soynlp` 패키지의 Noun Extractor을 학습시켰습니다. 각 주별로 어떤 신조어가 추출되었는지 확인하기 위해 주 별로 문서를 나누었습니다. 추출 시 사용한 환경은 아래와 같습니다.
- soynlp == 0.0.47
- LRNounExtractor_v2
- min_noun_score == 0.3
- min_noun_frequency == 2

이후, 중복된 단어를 제거하고 신조어로 저장될 기준을 만들었습니다.
- 동일한 주에 추출된 동일한 명사는 실제로 처음 그 단어가 사용된 날짜 기준으로 먼저 언급된 쪽을 선택
- 위 조건에 부합하지만 추출된 카테고리가 달라 실제로 동일한 명사임에도 불구하고 다르게 분류된 경우, 빈도수가 높은 카테고리에서 본 단어가 추출되었다고 가정
- 각 주별 최소 빈도수가 5 이상인 단어만 신조어로 분류

마지막으로, 준비해 둔 세종 코퍼스와 비교하면서 각 주별 신조어를 저장하였습니다. 결과의 일부를 빈도수 기준으로 정렬하였을 때 아래와 같은 결과가 나타납니다.

<img src="https://user-images.githubusercontent.com/25416425/71515586-506b7f00-28e7-11ea-80e3-d70da3a44960.png" width="550">

## Guide
### Usage guide

현재 버전에서는 미리 추출해 놓은 신조어 코퍼스를 다운로드 하는 것만 가능합니다. 추후 자동으로 사전을 업데이트 하는 코드를 추가할 예정입니다.
신조어 코퍼스는 두 가지 버전으로 제공됩니다. 본 저장소를 clone 해서 사용하시면 됩니다.

~~~
$ git clone https://github.com/jeongwookie/new_korean_words.git
~~~

### 신조어 코퍼스 JSON 
첫 번째는 key가 단어이고 이에 대응하는 value가 속성인 python dictionary 형태 입니다. `new_word_corpus_132799_key_noun.json` 이라는 이름으로 제공됩니다. 각 속성에 대해서는 아래 설명을 달아 놓았습니다.

- week : 해당 단어가 추출된 주
- frequency : 해당 단어가 해당 주에서 언급된 빈도수
- noun_score : soynlp의 noun extractor에서 제공하는 명사 수치
- sid : NAVER 뉴스 카테고리
- exact_date : 해당 단어가 문서 내에서 처음 발견된 날짜

간단하게 예시를 들어보면 아래와 같습니다.

~~~python
with open(FILE_PATH) as file: # 코퍼스를 다운로드 받은 곳을 FILE_PATH 으로 지정   
    NEW_WORDS = json.load(file)
    
print(NEW_WORDS["알파고"])
# {'week': '20160125',
#  'frequency': '19',
#  'noun_score': '1.0',
#  'sid': '106',
#  'exact_date': '20160128'}

print(NEW_WORDS["리그오브레전드"])
# {'week': '20111212',
#  'frequency': '15',
#  'noun_score': '1.0',
#  'sid': '105',
#  'exact_date': '20111212'}

print(NEW_WORDS["하스스톤"])
# {'week': '20131223',
#  'frequency': '6',
#  'noun_score': '1.0',
#  'sid': '105',
#  'exact_date': '20131223'}
~~~

### Mecab 사용자 사전 추가


