# Output

將擷取的內容轉成 ninjs3.0 JSON

### ninjs3.0 範例

Referance
https://github.com/iptc/newsinjson/blob/main/examples/3.0/20231008-anti-doping-measures-at-the-tour-down-under-will-be-the-toughest-ever-addb-ninjs-nitf.json
https://github.com/iptc/newsinjson/blob/main/examples/3.0/businesswire-newsml-20130731006140.json
https://www.iptc.org/std/ninjs/userguide/#_multimedia_example

```{dropdown} example metadata
{
"uri": "URL from warc",
"standard": {
"name": "ninjs",
"version": "3.0",
"schema": "https://www.iptc.org/std/ninjs/ninjs-schema_3.0.json"
},
"firstcreated": "2024-02-07",
"versioncreated": "2024-02-07",
"contentcreated": "2024-02-07",
"type": "text",
"language": "zh-Hant-TW"
"headlines": [
{
"role": "main",
"value": "This is Headline"
}
],
"subjects": [
{
"name": "Crime, law and justice"
}
],
"bodies": [
{
"role": "main",
"contentType": "text/plain",
"value": "body content "
}
],
"associations": [
{
"name": "mainpic",
"uri":"URL from warc photo",
"type":"picture",
"headlines" : [
{
"value": "picture's title"
}
],
"renditions" : [
{
"href" : "./ID/img/X.jpg (relative path)",
"contentType": "image/jpg",

                }
              ]
        },
        {
              "name": "firstpic",
              "uri":"URL from warc photo",
              "type":"picture",
              "headlines" : [
                {
                      "value": "picture's title"
                }
              ],
              "renditions" : [
                {
                  "href" : "./ID/img/X.jpg (relative path)",
                  "contentType": "image/jpg",

                }
              ]
        },
        {
              "name": "secondpic....",
              "uri":"URL from warc photo",
              "type":"picture",
              "headlines" : [
                {
                      "value": "picture's title"
                }
              ],
              "renditions" : [
                {
                  "href" : "./PID/img/X.jpg (relative path)",
                  "contentType": "image/jpg",

                }
              ]
        }

],
"by": "by whom",
"located": "place name",
"altids": [
{
"role": "internal",
"value": "TNT3L7GJIB7M3WEDXNIS3MYPGI"
}
]
}
```

```{dropdown} output metadata

  {
    "uri": "https://www.appledaily.com.tw/local/20220101/4C4Q3YRHTZGGBN2YAHLJH4CSQI/",
    "standard": {
      "name": "ninjs",
      "version": "3.0",
      "schema": "https://www.iptc.org/std/ninjs/ninjs-schema_3.0.json"
    },
    "firstcreated": "2022-01-01",
    "versioncreated": "2022-09-06",
    "contentcreated": "2022-01-01",
    "type": "text",
    "language": "zh-Hant-TW",
    "headlines": [
      {
        "role": "main",
        "value": "台中跨年夜街頭血戰逮5人！「國旗男」不敵雙刀客　星巴克顧客驚叫逃跑"
      }
    ],
    "subjects": [
      {
        "name": "local"
      }
    ],
    "bodies": [
      {
        "role": "main",
        "contentType": "text/plain",
        "value": "（新增：案情）\n跨年夜民眾喜迎2022年到來之際，台中市街頭又驚傳街頭砍殺事件，二幫人馬昨晚（31日）相約在北屯區協商債務，談判破裂後其中一方竟持刀械當街砍殺，造成二人刀傷送醫，遭砍一方就地拔起國旗反擊，仍不敵持刀一方猛攻，嚇得落荒而逃，警方獲報啟動快打部隊，循線逮捕5人到案，訊後依殺人未遂、妨害秩序罪移送，檢察官聲請羈押。\n由於事發地點位在星巴克前方，跨年的星巴克顧客隔著玻璃窗目睹這一幕，嚇到尖叫，紛紛閃躲快逃。\n根據影片內容顯示，翁姓男子（39歲）一夥人與王姓男子（40歲）等人，在北屯中清路二段星巴克咖啡前的人行道，爆發大亂鬥。\n翁男見王男跌坐地上，立即持刀朝王男狠砍一刀，王男同夥林男（43歲）見狀也朝對方丟擲物品反擊，翁男一方立即追上前，趁林男跌倒時，又趨前朝林男砍殺一刀。\n王男另名同夥見自己人被砍殺，竟不知從拿裡取得一面連接竿子的國旗，攻擊持刀的翁男，結果國旗與竿子分離，嚇得趕緊落跑，而站在馬路上的王男同夥，也在逃跑過程中，遭翁男一方丟擲物品擊中「仆街」，翁男一行人行兇後，立即搭乘一輛白色賓士車離去。\n轄區第五分局表示，12月31日晚間9點41分獲報，中清路二段星巴克咖啡館前發生打架案件，立即啟動快打部隊9警網18人馳赴，在現場發現血跡，但茲事份子已經散去，經調閱現場監視器，迅速逮捕3人，下午又循線查獲2名在逃共犯，查扣開山刀及柴刀。\n警方指出，翁男、林男（23歲）、吳男（27歲）等人，與王男、林男、吳男（30歲）、梁女（23歲）、蔡男（42歲）等5人，因一筆數萬元債務相約協商，在談判過程中持械鬥毆，造成王男及林男遭翁男持刀砍傷，自行前往台中榮總及中港澄清醫院就醫，所幸只有皮肉傷，並無大礙。\n針對有臉書社團PO出警察街頭逮捕攔查嫌犯畫面，警方澄清表示，案發當下快打部隊到場，立即根據目擊民眾提供的歹徒乘坐白色賓士車逃逸，立即展開攔查圍捕，經盤查與砍殺案無關。（地方中心王煌忠／台中報導）"
      }
    ],
    "associations": [
      {
        "name": "cover",
        "uri": "https://web.archive.org/web/20220906014810im_/https://static-arc.appledaily.com.tw/20220101/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/YPU5LSPMSNHYXO5CNQLWMH7X4U.png",
        "type": "picture",
        "headlines": [
          {
            "value": ""
          }
        ],
        "renditions": [
          {
            "href": "./images/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/4C4Q3YRHTZGGBN2YAHLJH4CSQI_cover_1.png",
            "contentType": "image/png"
          }
        ]
      },
      {
        "name": "other_1",
        "uri": "https://web.archive.org/web/20220906014810im_/https://static-arc.appledaily.com.tw/20220101/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/G2BCL5DN6FDUFD6YQURUKTPOHI.png",
        "type": "picture",
        "headlines": [
          {
            "value": ""
          }
        ],
        "renditions": [
          {
            "href": "./images/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/4C4Q3YRHTZGGBN2YAHLJH4CSQI_other_2.png",
            "contentType": "image/png"
          }
        ]
      },
      {
        "name": "other_2",
        "uri": "https://web.archive.org/web/20220906014810im_/https://static-arc.appledaily.com.tw/20220101/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/SDV6GYPQ6ZDVXJ3JTI7N42BSOM.jpg",
        "type": "picture",
        "headlines": [
          {
            "value": ""
          }
        ],
        "renditions": [
          {
            "href": "./images/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/4C4Q3YRHTZGGBN2YAHLJH4CSQI_other_3.jpg",
            "contentType": "image/jpeg"
          }
        ]
      },
      {
        "name": "other_3",
        "uri": "https://web.archive.org/web/20220906014810im_/https://static-arc.appledaily.com.tw/20220101/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/RYCPO2RUGZAAJNXMPDX6JMZUPE.png",
        "type": "picture",
        "headlines": [
          {
            "value": ""
          }
        ],
        "renditions": [
          {
            "href": "./images/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/4C4Q3YRHTZGGBN2YAHLJH4CSQI_other_4.png",
            "contentType": "image/png"
          }
        ]
      },
      {
        "name": "other_4",
        "uri": "https://web.archive.org/web/20220906014810im_/https://static-arc.appledaily.com.tw/20220101/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/HBUCUDM5PBHX3KRAYT225Z5GRY.png",
        "type": "picture",
        "headlines": [
          {
            "value": ""
          }
        ],
        "renditions": [
          {
            "href": "./images/4C4Q3YRHTZGGBN2YAHLJH4CSQI/img/4C4Q3YRHTZGGBN2YAHLJH4CSQI_other_5.png",
            "contentType": "image/png"
          }
        ]
      }
    ],
    "by": "",
    "located": "",
    "altids": [
      {
        "role": "internal",
        "value": "4C4Q3YRHTZGGBN2YAHLJH4CSQI"
      }
    ]
  }
```
