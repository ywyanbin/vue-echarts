<template>
  <div id="myChart" :style="{ width: '100%', height: '100vh', overflow: 'auto', backgroundColor: '#fff' }"></div>
</template>

<script>
import * as echarts from 'echarts';
import mock from './data';
export default {
  data() {
    return {
      data: [],
      links: [],
      seriesData: [
        {
          type: 'graph',
          hoverAnimation: false,
          layout: 'circular', // 自定义布局
          symbol: 'circle',
          symbolSize: 120,
          symbolOffset: [0, 0],
          focusNodeAdjacency: true,
          roam: true,
          zlevel: 1,
          zoom: 1,
          nodeScaleRatio: 1,
          label: {
            show: true,
            position: 'bottom',
            formatter: params => {
              const { data } = params;
              return `{c|${data.name}}`;
            },
            rich: {
              c: {
                color: '#000',
                fontSize: 14,
                fontWeight: 'bold'
              }
            }
          },
          edgeSymbol: ['circle', 'arrow'], // 边两端的标记类型，可以是一个数组分别指定两端，也可以是单个统一指定。这里设置为箭头
          edgeSymbolSize: [4, 8],
          edgeLabel: {
            fontSize: 12
          },
          itemStyle: {
            color: 'transparent'
          },
          data: [], // 节点数据
          links: [],
          lineStyle: {
            // 统一设置边的样式
            opacity: 0.9,
            width: 2, // 边的宽度
            curveness: 0.1 // 曲度
          }
        }
      ]
    };
  },
  mounted() {
    const _data = mock.data.Response.Result;

    const objClone = JSON.parse(JSON.stringify(_data));

    for (var i = 0; i < objClone.length; i++) {
      for (var key in objClone[i]) {
        const reg = /^[a-z]+$/;
        if (!reg.test(key)) {
          var lowerkey = key.toLowerCase();
          objClone[i][lowerkey] = objClone[i][key];
          delete objClone[i][key];
        }
      }
    }

    let edges = [];

    const newData = objClone.map(v => {
      if (v.edgelist.length > 0) {
        edges.push(...v.edgelist);
      }
      return {
        ...v
        // x: 0 + (i + 1) * 50,
        // y: (1 + Math.round(Math.random() * 5)) * 25
      };
    });

    const newLinks = edges.map(v => {
      return {
        source: v.SourceId,
        target: v.TargetId
      };
    });

    // console.log(newData);

    this.data = newData;
    this.links = newLinks;
    this.seriesData[0] = {
      ...this.seriesData[0],
      data: newData,
      links: newLinks
    };

    this.drawLine();
  },
  methods: {
    getPieSeries(graphData, chart) {
      const modal = chart.getModel().getSeriesByIndex(0);
      const info = modal.getData();
      const zoom = chart._coordSysMgr._coordinateSystems[0]._zoom;

      const l = [];
      for (let i = 0; i < info._nameList.length; i++) {
        l.push({
          name: info._nameList[i],
          x: info._itemLayouts[i][0],
          y: info._itemLayouts[i][1]
        });
      }

      l.map(item => {
        graphData.forEach(v => {
          if (item.name === v.name) {
            v.x = item.x;
            v.y = item.y;
          }
        });
      });

      return graphData.map(item => {
        var center = chart.convertToPixel({ seriesIndex: 0 }, [item.x, item.y]);
        if (item.reqsuccessrate) {
          return {
            focusNodeAdjacency: true,
            name: item.name,
            hoverAnimation: false,
            id: item.id,
            zlevel: 0,
            type: 'pie',
            startAngle: 90, //起始角度
            clockwise: true, //生长方向，是否顺时针
            radius: [55 * zoom, 60 * zoom],
            // radius: ["10%", "12%"],
            center: center,
            label: {
              show: true,
              position: 'center',
              formatter: () => {
                if (item.reqavgduration && item.reqavgqty) {
                  return [`{a|Avg. ${item.reqavgduration}ms}`, `{b|${item.reqavgqty} t/min}`].join('\n');
                }
                return '';
              },
              rich: {
                a: {
                  color: '#000',
                  fontSize: 14,
                  fontWeight: 'bold',
                  lineHeight: 25,
                  align: 'center'
                },
                b: {
                  color: '#000',
                  fontSize: 14,
                  fontWeight: 'bold',
                  lineHeight: 25,
                  align: 'center'
                }
              }
            },
            data: [
              {
                value: Number(item.reqsuccessrate),
                itemStyle: { color: '#009D04' }
              },
              {
                value: Number(1 - Number(item.reqsuccessrate)),
                itemStyle: { color: '#F5A623' }
              }
            ]
          };
        } else {
          return {
            focusNodeAdjacency: true,
            name: item.name,
            hoverAnimation: false,
            id: item.id,
            zlevel: 0,
            type: 'pie',
            radius: [55 * zoom, 60 * zoom],
            // radius: ["10%", "12%"],
            center: center,
            label: {
              show: false
            },
            itemStyle: {
              color: '#999'
            },
            data: [{ value: 0 }]
          };
        }
      });
    },
    drawLine() {
      let myChart = echarts.init(document.getElementById('myChart'));

      const option = {
        title: {
          text: 'Graph + Pie'
        },
        toolbox: {
          show: true,
          feature: {
            // restore: {},
            myFullScreen: {
              show: true,
              title: '全屏',
              icon: `image://${require('@/assets/screen.png')}`,
              onclick: () => {
                const element = document.getElementById('myChart');
                if (element.requestFullScreen) {
                  // HTML W3C 提议
                  element.requestFullScreen();
                } else if (element.msRequestFullscreen) {
                  // IE11
                  element.msRequestFullScreen();
                } else if (element.webkitRequestFullScreen) {
                  // Webkit (works in Safari5.1 and Chrome 15)
                  element.webkitRequestFullScreen();
                } else if (element.mozRequestFullScreen) {
                  // Firefox (works in nightly)
                  element.mozRequestFullScreen();
                }
                // 退出全屏
                if (element.requestFullScreen) {
                  document.exitFullscreen();
                } else if (element.msRequestFullScreen) {
                  document.msExitFullscreen();
                } else if (element.webkitRequestFullScreen) {
                  document.webkitCancelFullScreen();
                } else if (element.mozRequestFullScreen) {
                  document.mozCancelFullScreen();
                }
              }
            }
          }
        },
        tooltip: { show: false },
        series: this.seriesData
      };

      myChart.setOption(option);

      myChart.setOption({
        series: this.getPieSeries(this.data, myChart)
      });

      window.onresize = () => {
        myChart && myChart.resize();

        myChart.setOption({
          series: this.getPieSeries(this.data, myChart)
        });
      };

      myChart.on('mouseover', params => {
        let option = myChart.getOption();
        // 复制一份转换后的节点内容
        const _option = option.series.slice(1);

        if (params.dataType === 'edge') {
          // 获取边前后的节点id
          const source = params.data.source;
          const target = params.data.target;

          const nodeIds = [].concat(source, target);

          // 过滤，获取没有关联的节点信息
          const node = _option.filter(v => !nodeIds.includes(v.id));

          // 设置节点的透明度
          node.map(item => {
            const i = option.series.findIndex(v => v.id === item.id);
            option.series[i].itemStyle = {
              opacity: 0.1
            };
          });

          myChart.setOption(option);
        } else if (params.dataType === 'node') {
          // 获取当前选择节点的id值
          const index = option.series.findIndex(v => v.name === params.name);
          const nodeId = option.series[index].id;
          // 获取边的信息
          const links = option.series[0].links;

          // 获取当前节点兄弟节点的id
          const _links = links.filter(v => v.source === nodeId || v.target === nodeId);

          if (_links.length > 0) {
            const sourceId = _links.map(v => v.source);
            const targetId = _links.map(v => v.target);
            const otherId = Array.from(new Set([...sourceId, ...targetId]));
            // 过滤，获取没有关联的节点信息
            const node = _option.filter(v => !otherId.includes(v.id));

            // 设置节点的透明度
            node.map(item => {
              const i = option.series.findIndex(v => v.id === item.id);
              option.series[i].itemStyle = {
                opacity: 0.1
              };
            });
          } else {
            const node = _option.filter(v => v.id !== nodeId);
            // 设置节点的透明度
            node.map(item => {
              const i = option.series.findIndex(v => v.id === item.id);
              option.series[i].itemStyle = {
                opacity: 0.1
              };
            });
          }

          myChart.setOption(option);
        }
      });

      myChart.on('mouseout', () => {
        const option = myChart.getOption();
        // 鼠标移出后，opacity恢复
        for (let i = 1; i < option.series.length; i++) {
          option.series[i].itemStyle = {
            opacity: 1
          };
        }
        myChart.setOption(option);
      });

      myChart.on('graphRoam', () => {
        myChart.setOption({
          series: this.getPieSeries(this.data, myChart)
        });
        myChart && myChart.resize();
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
