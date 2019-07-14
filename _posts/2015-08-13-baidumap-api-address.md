---
layout: post
title:  "利用Baidu API正向和反向解析地址"
tags: [Server]
---

* 在开发过程中，假如遇到了需要计算两地址之间的距离，或者是需要在地图上定位某个地点，之类之类的需求，就会用到百度地图或者高德地图提供的API去解析一个地址，包括正向解析和反向解析。正向解析是输入一个地址，返回一个经纬度，相当于我们在搜索框搜索一个地址然后在地图上打个点。反向解析则是输入一个经纬度，返回一个地址的列表，这个列表是表示这个经纬度附近可能有的东西。

* 百度地图，解析过程实际上就是向百度的服务器接口通过一个特定格式的URL发送一个带有数据信息的Http请求，然后获得一个带有json数据的Http相应。所以，开发者需要完成的工作，就是把json数据包装到合适的url，以及接受到json数据之后解析一下。

### 代码(正向解析和反向解析直接封装到类BaiduGeocoder里面)

```
/**
 * Created by charlie on 15/8/7.
 */
public class BaiduGeocoder {

    private static String ak = "KaK3WhH5p6ali0IM5vi9sRlk";

    //    把代码中的ak值（红色字部分）更改为你自己的ak值，在百度地图API中注册一下就有。
    //    调用方式：
    //    Map<String,String> map=BaiduGeocoder.geoCoding(“广州市");
    //    System.out.println("经度："+map.get("lng")+"---纬度："+map.get("lat"));

    public static Map<String,String> geoCoding(String address){

        String url = "http://api.map.baidu.com/geocoder/v2/?"
                +"address=" +address
                +"&output=json"
                +"&ak="+ak
                +"&callback=showLocation";

        String json = loadJSON(url);
        json = json.substring(27,json.length()-1);
        JSONObject obj = JSONObject.parseObject(json);

        Map<String,String> map = new HashMap<>();
        if(obj.get("status").toString().equals("0")){
            Double lng=obj.getJSONObject("result").getJSONObject("location").getDouble("lng");
            Double lat=obj.getJSONObject("result").getJSONObject("location").getDouble("lat");
            map.put("lng", lng.toString());
            map.put("lat", lat.toString());
        }else{
            System.out.println("未找到相匹配的经纬度！");
        }
        return map;
    }
    
    //    调用方式：
    //    Map<String,String> map=BaiduGeocoder.reverseGeoCoding(”39.1“,“116.1”);     //    System.out.println("经度："+map.get("lng")+"---纬度："+map.get("lat"));     public static Map<String, String> reverseGeoCoding(String lng, String lat) {         String url = "http://api.map.baidu.com/geocoder/v2/?"                 + "ak=" + ak
                + "&callback=renderReverse"
                + "&location=" + lat + "," + lng
                + "&output=json&pois=1";

        String json = loadJSON(url);
        json = json.substring(29,json.length()-1);
        JSONObject obj = JSONObject.parseObject(json);

        Map<String,String> map = new HashMap<>();
        if(obj.get("status").toString().equals("0")){
            String address = obj.getJSONObject("result").getString("formatted_address");
            String province = obj.getJSONObject("result").getJSONObject("addressComponent").getString("province");
            String city = obj.getJSONObject("result").getJSONObject("addressComponent").getString("city");
            map.put("address",address);
            map.put("province",province);
            map.put("city",city);
        }else{
            System.out.println("未找到匹配经纬度的地址");
        }
        return map;
    }


    public static String loadJSON (String url) {
        StringBuilder json = new StringBuilder();
        try {
            URL oracle = new URL(url);
            URLConnection urlConnection = oracle.openConnection();
            BufferedReader in = new BufferedReader(new InputStreamReader(
                    urlConnection.getInputStream()));
            String inputLine;
            while ( (inputLine = in.readLine()) != null) {
                json.append(inputLine);
            }
            in.close();
        } catch (MalformedURLException e) {
            System.out.println("malformed exception");
        } catch (IOException e) {
            System.out.println("io exception");
        }

        return json.toString();
    }
}
```