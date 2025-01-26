# 地图飞线

```tsx
import React, { useEffect, useRef } from 'react';
import * as echarts from 'echarts';
import china from '@/assets/china.json';

const Page: React.FC = () => {
  const chinaMapRef = useRef(null);
  // 方法
  const initEcharts = () => {
    /* 起点坐标 */
    const lineFrom = [
      { name: '重庆市', latlng: [106.33, 29.35] },
      { name: '北京市', latlng: [116.07, 40.11] },
    ];

    /* 终点坐标 */
    const lineTo = [
      { name: '盘锦市', latlng: [120.93141287481329, 40.93448132827849] },
      { name: '沧州市', latlng: [116.71809759843096, 37.96769678343516] },
      { name: '东营市', latlng: [118.29234782217573, 37.44294670885357] },
      { name: '大连市', latlng: [121.26593157813807, 38.886009413952934] },
      { name: '沈阳市', latlng: [122.220947193165, 41.64094730550629] },
      { name: '北京市', latlng: [116.07673639616456, 40.110426254643315] },
      { name: '白银市', latlng: [101.09220648866805, 36.568363251217576] },
      { name: '石家庄市', latlng: [115.20215293852858, 38.886009413952934] },
    ];

    /* 飞线数据 */
    let linesData: any = [];

    lineFrom.forEach((from) => {
      lineTo.forEach((to) => {
        linesData.push({
          coords: [from.latlng, to.latlng],
          fromName: from.name,
          toName: to.name,
          lineStyle: {
            color: '#FFE747',
            curveness: 0.2,
          },
        });
      });
    });

    /* 起点数据 */
    const fromPointsData = lineFrom.map((from) => {
      return {
        value: from.latlng,
        name: from.name,
      };
    });

    /* 终点数据 */
    const toPointsData = lineTo.map((row) => {
      return {
        value: row.latlng,
        name: row.name,
        lineStyle: {
          color: '#FFE747',
        },
      };
    });

    /* 飞线-series */
    const lineSeries = [
      // 飞线
      {
        type: 'lines',
        zlevel: 5,
        effect: {
          show: true,
          period: 5, //箭头指向速度，值越小速度越快
          trailLength: 0, //特效尾迹长度[0,1]值越大，尾迹越长重
          symbol: 'arrow', //ECharts 提供的标记类型包括 'circle', 'rect', 'roundRect', 'triangle', 'diamond', 'pin', 'arrow'
          symbolSize: 8, //图标大小
        },
        lineStyle: {
          color: '#FFE747',
          type: 'dashed',
          width: 2, //尾迹线条宽度
          opacity: 1, //尾迹线条透明度
          curveness: 0.3, //尾迹线条曲直度
        },
        data: linesData,
        markPoint: {
          symbol: 'circle', //ECharts 提供的标记类型包括 'circle', 'rect', 'roundRect', 'triangle', 'diamond', 'pin', 'arrow'
          symbolSize: 8, //图标大小
        },
      },
      //起点
      {
        type: 'effectScatter',
        coordinateSystem: 'geo',
        zlevel: 6,
        rippleEffect: {
          //涟漪特效
          period: 4, //动画时间，值越小速度越快
          brushType: 'stroke', //波纹绘制方式 stroke, fill
          scale: 4, //波纹圆环最大限制，值越大波纹越大
        },
        label: {
          show: false,
          position: 'right', //显示位置
          offset: [5, 0], //偏移设置
          formatter: '{b}', //圆环显示文字
          color: 'red',
        },
        symbol: 'circle',
        symbolSize: function (val) {
          return 10; //圆环大小
        },
        itemStyle: {
          show: false,
          color: '#ff8400',
        },
        data: fromPointsData,
      },
      // 终点
      {
        type: 'effectScatter',
        coordinateSystem: 'geo',
        zlevel: 6,
        rippleEffect: {
          //涟漪特效
          period: 4, //动画时间，值越小速度越快
          brushType: 'stroke', //波纹绘制方式 stroke, fill
          scale: 4, //波纹圆环最大限制，值越大波纹越大
        },
        label: {
          show: false,
          position: 'right', //显示位置
          offset: [5, 0], //偏移设置
          formatter: '{b}', //圆环显示文字
          color: 'red',
        },
        symbol: 'circle',
        symbolSize: function (val) {
          return 6; //圆环大小
        },
        itemStyle: {
          show: false,
          color: '#befaff',
        },
        data: toPointsData,
      },
    ];

    //注册地图，这个特别重要
    echarts.registerMap('china', china);

    const myChart = echarts.init(chinaMapRef.current);
    //echart 配制option
    const options = {
      tooltip: {
        show: true,
        triggerOn: 'mousemove', //mousemove、click
        padding: [4, 8],
        borderWidth: 1,
        borderColor: '#409eff',
        backgroundColor: 'rgba(255,255,255,0.7)',
        textStyle: {
          color: '#000000',
          fontSize: 13,
        },
        formatter: function (e) {
          return e.name;
        },
      },
      geo: [
        {
          map: 'china',
          z: 3,
          zoom: 1.2,
          aspectScale: 0.85,
          roam: false,
          top: '10%',
          layoutSize: '100%', //保持地图宽高比
          regions: [
            {
              // 隐藏南海诸岛,因为顶层已经添加过了
              name: '南海诸岛',
              itemStyle: {
                borderWidth: 0.5,
                shadowBlur: 0,
                borderColor: '#61aacb',
                areaColor: '#104584',
              },
            },
          ],
          itemStyle: {
            borderColor: '#c8feff',
            borderWidth: 0.5,
            shadowBlur: 3,
            shadowColor: '#66edff',
            areaColor: '#0862db',
          },
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              borderWidth: 1,
              areaColor: '#2da9ff',
            },
            label: {
              show: false,
              color: '#ffffff',
            },
          },
          select: {
            itemStyle: {
              shadowBlur: 10,
              borderWidth: 1,
              areaColor: '#2da9ff',
            },
            label: {
              color: '#ffffff',
            },
          },
        },
      ],
      series: [
        // 地图
        {
          type: 'map',
          geoIndex: 0,
          data: [],
        },
        ...lineSeries,
      ],
    };
    myChart.setOption(options);
  };

  useEffect(() => {
    initEcharts();
  }, []);
  return (
    <>
      <div className="h-screen w-screen" ref={chinaMapRef}></div>
    </>
  );
};

export default Page;
```