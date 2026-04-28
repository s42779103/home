<template>
  <div class="weather" v-if="weatherData.city && weatherData.weather">
    <span>{{ weatherData.city }}&nbsp;</span>
    <span>{{ weatherData.weather }}&nbsp;</span>
    <span>{{ weatherData.temperature }}℃</span>
    <span class="sm-hidden">
      &nbsp;{{
        weatherData.winddirection?.endsWith("风")
          ? weatherData.winddirection
          : weatherData.winddirection + "风"
      }}&nbsp;
    </span>
    <span class="sm-hidden">{{ weatherData.windpower }}&nbsp;级</span>
  </div>
  <div class="weather" v-else>
    <span>天气获取失败</span>
  </div>
</template>

<script setup>
import { getWeather } from "@/api";
import { Error } from "@icon-park/vue-next";

// 城市英文名 -> 中文名映射
const cityZhMap = {
  // 直辖市
  "Beijing": "北京",
  "Tianjin": "天津",
  "Shanghai": "上海",
  "Chongqing": "重庆",
  // 河北
  "Shijiazhuang": "石家庄",
  "Tangshan": "唐山",
  "Qinhuangdao": "秦皇岛",
  "Handan": "邯郸",
  "Baoding": "保定",
  "Langfang": "廊坊",
  "Cangzhou": "沧州",
  "Chengde": "承德",
  "Zhangjiakou": "张家口",
  "Xingtai": "邢台",
  "Hengshui": "衡水",
  // 山西
  "Taiyuan": "太原",
  "Datong": "大同",
  "Changzhi": "长治",
  "Linfen": "临汾",
  "Jinzhong": "晋中",
  "Yuncheng": "运城",
  // 内蒙古
  "Hohhot": "呼和浩特",
  "Baotou": "包头",
  "Chifeng": "赤峰",
  "Ordos": "鄂尔多斯",
  "Hulunbuir": "呼伦贝尔",
  // 辽宁
  "Shenyang": "沈阳",
  "Dalian": "大连",
  "Anshan": "鞍山",
  "Fushun": "抚顺",
  "Benxi": "本溪",
  "Jinzhou": "锦州",
  "Yingkou": "营口",
  "Dandong": "丹东",
  "Panjin": "盘锦",
  "Liaoyang": "辽阳",
  "Huludao": "葫芦岛",
  // 吉林
  "Changchun": "长春",
  "Jilin": "吉林",
  "Siping": "四平",
  "Yanji": "延吉",
  // 黑龙江
  "Harbin": "哈尔滨",
  "Daqing": "大庆",
  "Qiqihar": "齐齐哈尔",
  "Mudanjiang": "牡丹江",
  "Jiamusi": "佳木斯",
  "Suihua": "绥化",
  "Heihe": "黑河",
  // 江苏
  "Nanjing": "南京",
  "Suzhou": "苏州",
  "Wuxi": "无锡",
  "Changzhou": "常州",
  "Nantong": "南通",
  "Zhenjiang": "镇江",
  "Yangzhou": "扬州",
  "Yancheng": "盐城",
  "Lianyungang": "连云港",
  "Xuzhou": "徐州",
  "Huaian": "淮安",
  "Taizhou": "泰州",
  "Suqian": "宿迁",
  // 浙江
  "Hangzhou": "杭州",
  "Ningbo": "宁波",
  "Wenzhou": "温州",
  "Shaoxing": "绍兴",
  "Jiaxing": "嘉兴",
  "Huzhou": "湖州",
  "Jinhua": "金华",
  "Zhoushan": "舟山",
  "Taizhou Zhejiang": "台州",
  "Quzhou": "衢州",
  "Lishui": "丽水",
  // 安徽
  "Hefei": "合肥",
  "Wuhu": "芜湖",
  "Bengbu": "蚌埠",
  "Huainan": "淮南",
  "Ma'anshan": "马鞍山",
  "Anqing": "安庆",
  "Chuzhou": "滁州",
  "Fuyang": "阜阳",
  // "Suzhou": "宿州", (已移除，与江苏苏州英文名冲突)
  "Xuancheng": "宣城",
  "Lu'an": "六安",
  "Tongling": "铜陵",
  "Chizhou": "池州",
  "Huangshan": "黄山",
  // 福建
  "Fuzhou": "福州",
  "Xiamen": "厦门",
  "Quanzhou": "泉州",
  "Zhangzhou": "漳州",
  "Putian": "莆田",
  "Nanping": "南平",
  "Longyan": "龙岩",
  "Ningde": "宁德",
  // 江西
  "Nanchang": "南昌",
  "Jiujiang": "九江",
  "Ganzhou": "赣州",
  "Jingdezhen": "景德镇",
  "Pingxiang": "萍乡",
  "Yichun": "宜春",
  "Shangrao": "上饶",
  // 山东
  "Jinan": "济南",
  "Qingdao": "青岛",
  "Yantai": "烟台",
  "Weihai": "威海",
  "Zibo": "淄博",
  "Weifang": "潍坊",
  "Linyi": "临沂",
  "Jining": "济宁",
  "Tai'an": "泰安",
  "Dongying": "东营",
  "Rizhao": "日照",
  "Binzhou": "滨州",
  "Liaocheng": "聊城",
  "Heze": "菏泽",
  "Zaozhuang": "枣庄",
  "Dezhou": "德州",
  // 河南
  "Zhengzhou": "郑州",
  "Luoyang": "洛阳",
  "Kaifeng": "开封",
  "Xinxiang": "新乡",
  "Anyang": "安阳",
  "Jiaozuo": "焦作",
  "Nanyang": "南阳",
  "Xuchang": "许昌",
  "Pingdingshan": "平顶山",
  "Shangqiu": "商丘",
  "Zhoukou": "周口",
  "Luohe": "漯河",
  "Puyang": "濮阳",
  "Hebi": "鹤壁",
  "Sanmenxia": "三门峡",
  "Xinyang": "信阳",
  "Zhumadian": "驻马店",
  // 湖北
  "Wuhan": "武汉",
  "Yichang": "宜昌",
  "Xiangyang": "襄阳",
  "Jingzhou": "荆州",
  "Huangshi": "黄石",
  "Shiyan": "十堰",
  "Xiaogan": "孝感",
  "Huanggang": "黄冈",
  "Xianning": "咸宁",
  "Jingmen": "荆门",
  "Ezhou": "鄂州",
  "Suizhou": "随州",
  // 湖南
  "Changsha": "长沙",
  "Zhuzhou": "株洲",
  "Xiangtan": "湘潭",
  "Hengyang": "衡阳",
  "Yueyang": "岳阳",
  "Changde": "常德",
  "Shaoyang": "邵阳",
  "Chenzhou": "郴州",
  "Yiyang": "益阳",
  "Loudi": "娄底",
  "Huaihua": "怀化",
  "Yongzhou": "永州",
  "Zhangjiajie": "张家界",
  // 广东
  "Guangzhou": "广州",
  "Shenzhen": "深圳",
  "Zhuhai": "珠海",
  "Dongguan": "东莞",
  "Foshan": "佛山",
  "Zhongshan": "中山",
  "Huizhou": "惠州",
  "Shantou": "汕头",
  "Jieyang": "揭阳",
  "Zhaoqing": "肇庆",
  "Jiangmen": "江门",
  "Meizhou": "梅州",
  "Shaoguan": "韶关",
  "Maoming": "茂名",
  "Zhanjiang": "湛江",
  "Yangjiang": "阳江",
  "Qingyuan": "清远",
  "Shanwei": "汕尾",
  "Heyuan": "河源",
  "Yunfu": "云浮",
  "Chaozhou": "潮州",
  // 广西
  "Nanning": "南宁",
  "Guilin": "桂林",
  "Liuzhou": "柳州",
  "Beihai": "北海",
  "Wuzhou": "梧州",
  "Yulin": "玉林",
  "Qinzhou": "钦州",
  "Fangchenggang": "防城港",
  "Guigang": "贵港",
  "Baise": "百色",
  "Hechi": "河池",
  "Laibin": "来宾",
  "Hezhou": "贺州",
  "Chongzuo": "崇左",
  // 海南
  "Haikou": "海口",
  "Sanya": "三亚",
  "Danzhou": "儋州",
  // 四川
  "Chengdu": "成都",
  "Mianyang": "绵阳",
  "Deyang": "德阳",
  "Yibin": "宜宾",
  "Nanchong": "南充",
  "Luzhou": "泸州",
  "Dazhou": "达州",
  "Leshan": "乐山",
  "Zigong": "自贡",
  "Panzhihua": "攀枝花",
  "Guangyuan": "广元",
  "Suining": "遂宁",
  "Neijiang": "内江",
  "Guangan": "广安",
  "Meishan": "眉山",
  "Ya'an": "雅安",
  "Bazhong": "巴中",
  "Ziyang": "资阳",
  "Xichang": "西昌",
  // 贵州
  "Guiyang": "贵阳",
  "Zunyi": "遵义",
  "Anshun": "安顺",
  "Bijie": "毕节",
  "Tongren": "铜仁",
  // 云南
  "Kunming": "昆明",
  "Dali": "大理",
  "Lijiang": "丽江",
  "Yuxi": "玉溪",
  "Qujing": "曲靖",
  "Baoshan": "保山",
  "Zhaotong": "昭通",
  "Puer": "普洱",
  "Chuxiong": "楚雄",
  "Gejiu": "个旧",
  // 西藏
  "Lhasa": "拉萨",
  "Shigatse": "日喀则",
  // 陕西
  "Xi'an": "西安",
  "Xianyang": "咸阳",
  "Baoji": "宝鸡",
  "Weinan": "渭南",
  "Hanzhong": "汉中",
  "Yan'an": "延安",
  // "Yulin": "榆林", (已移除，与广西玉林英文名冲突)
  "Ankang": "安康",
  "Shangluo": "商洛",
  // 甘肃
  "Lanzhou": "兰州",
  "Tianshui": "天水",
  "Jiuquan": "酒泉",
  "Zhangye": "张掖",
  "Wuwei": "武威",
  "Jiayuguan": "嘉峪关",
  "Pingliang": "平凉",
  "Qingyang": "庆阳",
  // 青海
  "Xining": "西宁",
  "Golmud": "格尔木",
  // 宁夏
  "Yinchuan": "银川",
  "Shizuishan": "石嘴山",
  "Wuzhong": "吴忠",
  // 新疆
  "Urumqi": "乌鲁木齐",
  "Karamay": "克拉玛依",
  "Turpan": "吐鲁番",
  "Kashgar": "喀什",
  "Yining": "伊宁",
  "Aksu": "阿克苏",
  "Hami": "哈密",
  "Korla": "库尔勒",
  "Changji": "昌吉",
  // 港澳台
  "Taipei": "台北",
  "Kaohsiung": "高雄",
  "Taichung": "台中",
  "Tainan": "台南",
  "Taoyuan": "桃园",
  "Hong Kong": "香港",
  "Macau": "澳门",
};

const cityZh = (en) => cityZhMap[en] || en;
const windDirZh = (en) => windDirZhMap[en] || en;

const windDirZhMap = {
  "N": "北",
  "NNE": "东北偏北",
  "NE": "东北",
  "ENE": "东北偏东",
  "E": "东",
  "ESE": "东南偏东",
  "SE": "东南",
  "SSE": "东南偏南",
  "S": "南",
  "SSW": "西南偏南",
  "SW": "西南",
  "WSW": "西南偏西",
  "W": "西",
  "WNW": "西北偏西",
  "NW": "西北",
  "NNW": "西北偏北",
};

// 天气数据
const weatherData = reactive({
  city: null,
  weather: null,
  temperature: null,
  winddirection: null,
  windpower: null,
});

// 获取天气数据
const getWeatherData = async () => {
  const data = await getWeather();
  if (!data.current_condition) throw "天气接口返回异常";
  const current = data.current_condition[0];
  const area = data.nearest_area?.[0]?.areaName?.[0]?.value || "未知地区";
  weatherData.city = cityZh(area);
  weatherData.weather = current.lang_zh?.[0]?.value || current.weatherDesc?.[0]?.value || "未知";
  weatherData.temperature = current.temp_C;
  weatherData.winddirection = windDirZh(current.winddir16Point);
  const speedKmh = +current.windspeedKmph;
  weatherData.windpower = Math.round((speedKmh / 3.01) ** 0.67) + "";
};

// 报错信息
const onError = (message) => {
  ElMessage({
    message,
    icon: h(Error, {
      theme: "filled",
      fill: "#efefef",
    }),
  });
  console.error(message);
};

onMounted(() => {
  getWeatherData().catch((e) => {
    console.error("天气获取失败:", e);
    onError("天气信息获取失败");
  });
});
</script>
