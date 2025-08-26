# Output

將擷取的內容轉成 ninjs3.0

### ninjs3.0 範例

Referance
https://github.com/iptc/newsinjson/blob/main/examples/3.0/20231008-anti-doping-measures-at-the-tour-down-under-will-be-the-toughest-ever-addb-ninjs-nitf.json
https://github.com/iptc/newsinjson/blob/main/examples/3.0/businesswire-newsml-20130731006140.json
https://www.iptc.org/std/ninjs/userguide/#_multimedia_example

```
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

```

```
