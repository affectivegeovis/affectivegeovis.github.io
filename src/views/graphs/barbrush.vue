<template>
  <div ref="matrixTable" style="height: 100%; width: 100%;min-width: 300px;"></div>
</template>

<script>
import * as d3 from 'd3';

export default {
  name: 'TaxoMatrixTable',
  props: {
    selectedProjectName: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      techHierarchy: [],
      projects: [],
      techMap: new Map(),
      margin: { top: 10, right: 10, bottom: 10, left: 200 }
    }
  },
  watch: {
    selectedProjectName(newVal) {
      this.highlightProject(newVal);
      console.log("=====",newVal)
    }
  },
  methods: {
    highlightProject(projectName) {
      if (!projectName) return;
      
      // 找到项目对应的行（使用模糊匹配）
      const projectIndex = this.projects.findIndex(p => {
        // 如果完全匹配，直接返回
        if (p.name === projectName) return true;
        
        // 如果 barbrush 中的名称是 Service 中名称的一部分
        if (projectName.includes(p.name)) return true;
        
        // 如果 Service 中的名称是 barbrush 中名称的一部分
        if (p.name.includes(projectName)) return true;
        
        return false;
      });
      
      if (projectIndex === -1) return;
      
      // 获取SVG元素
      const svg = d3.select(this.$refs.matrixTable).select('svg');
      if (svg.empty()) return;
      
      // 移除之前的高亮
      svg.selectAll('.highlighted-row').remove();
      
      // 添加新的高亮
      const g = svg.select('g');
      const headerHeight = this.$refs.matrixTable.clientHeight / 60 * 15;
      const cellHeight = this.$refs.matrixTable.clientHeight / 85;
      
      // 绘制高亮背景
      g.append('rect')
        .attr('class', 'highlighted-row')
        .attr('x', -this.margin.left)
        .attr('y', headerHeight / 5 * 4 + projectIndex * cellHeight + cellHeight*1.5)
        .attr('width', this.$refs.matrixTable.clientWidth)
        .attr('height', cellHeight)
        .attr('fill', 'rgba(58, 52, 52, 0.1)')
        .attr('stroke', 'rgba(58, 52, 52, 0.5)')
        .attr('stroke-width', 1);
    },
    
    // 修改点击事件处理函数，也使用模糊匹配
    handleCellClick(projectName) {
      // 找到最匹配的项目名称
      const matchedProject = this.projects.find(p => {
        if (p.name === projectName) return true;
        if (projectName.includes(p.name)) return true;
        if (p.name.includes(projectName)) return true;
        return false;
      });
      
      if (matchedProject) {
        this.$emit('project-selected', matchedProject.name);
        console.log(matchedProject.name);
      }
    },
    
    getColorForTech(techName) {
      // 确保 techName 是一个有效的字符串
      if (!techName) return '#ffffff'; // 默认为白色
      
      techName = techName.toLowerCase();
      
      // 为不同技术类别分配颜色
      const colorMap = {
        'individual': '#F3E3E9',
        'group': '#E192AF',
        'residential': '#AEAAAA',
        'not specified or anyone': '#AEAAAA',
        'special status': '#AEAAAA',
        'traveler': '#AEAAAA',
        'cultural/racial': '#AEAAAA',
        'gender': '#AEAAAA',
        'age': '#AEAAAA',
        'class': '#AEAAAA',
        'world': '#204E78',
        'country': '#2F75B5',
        'city': '#9BC2E6',
        'non-urban/rural': '#BDD7EE',
        'architecture/site': '#DDEBF7',
        'physical': '#203764',
        'memorized': '#8EA9DB',
        'psychological': '#D8E1F2',
        'inherent emotional attitude': '#EBE9E3',
        'personal experience': '#EBE9E3',
        'collaborative activity': '#F2E2D9',
        'embodied movement': '#F2E2D9',
        'interview': '#F2E2D9',
        'drawing/sketching': '#F2E2D9',
        'fieldwork and ethnography': '#F2E2D9',
        'video recording/taking pictures': '#F2E2D9',
        'posting sticky notes': '#F2E2D9',
        'prepared toolkit': '#F2E2D9',
        'hardware': '#F2E2D9',
        'historical material mining': '#F2E2D9',
        'system/platform development': '#F2E2D9',
        'questionnaire and survey': '#F2E2D9',
        'statis image/painting': '#E7E6E6',
        'event': '#E7E6E6',
        'interactive interface': '#E7E6E6',
        'artifact': '#E7E6E6',
        'performance': '#E7E6E6',
        'video': '#E7E6E6',
        'installation': '#E7E6E6',
        'color': '#D1DFDA',
        'image/photo': '#D1DFDA',
        'symbol/metaphor': '#D1DFDA',
        'tangible material/texture': '#D1DFDA',
        'shape': '#D1DFDA',
        'sound': '#D1DFDA',
        'style': '#D1DFDA',
        'animation/simulation': '#D1DFDA',
        'layout': '#D1DFDA',
        'typeface': '#D1DFDA',
        'annotated content/associated stories': '#BFDFE0',
        'collage': '#BFDFE0',
        'body language': '#BFDFE0',
        'wording/phrasing': '#BFDFE0',
        'personalized visual': '#BFDFE0'
      };
      
      return colorMap[techName] || '#ffffff'; // 默认为白色
    },
    
    async drawTaxoMatrix() {
      try {
        // 加载 CSV 文件
        const taxoData = await d3.csv('taxo.csv');
        const codingData = await d3.csv('sorted_coding.csv');
        
        // 处理 taxo 数据
        this.techHierarchy = taxoData.map(d => ({
          tech: d.tech,
          id: +d.id,
          dim: d.dim,
          parent: d.parent,
          subparent: d.subparent,
          count: +d.count
        }));
        
        // 创建 techMap 用于快速查找
        this.techHierarchy.forEach(tech => {
          this.techMap.set(tech.tech, tech);
        });
        
        // 处理 coding 数据
        this.projects = codingData.map(project => {
          const techUsage = {};
          Object.keys(project).forEach(key => {
            if (key !== 'name') {
              const value = +project[key];
              if (value === 1) {
                // 将键名转换为与 tech.tech 一致的格式
                const formattedKey = key.replace(/\s+/g, ' '); // 确保键名中的空格一致
                techUsage[formattedKey] = value;
              }
            }
          });
          return {
            name: project.name,
            techUsage: techUsage
          };
        });

        // 创建 SVG 容器
        const width = this.$refs.matrixTable.clientWidth;
        const height = this.$refs.matrixTable.clientHeight;
        const innerWidth = width - this.margin.left - this.margin.right;
        const innerHeight = height - this.margin.top - this.margin.bottom;
        
        const svg = d3.select(this.$refs.matrixTable)
          .append('svg')
          .attr('width', '100%')
          .attr('height', '100%')
          .attr('viewBox', `0 0 ${width} ${height}`)
          .attr('preserveAspectRatio', 'xMidYMid meet');
        
        const g = svg.append('g')
          .attr('transform', `translate(${this.margin.left},${this.margin.top})`);
        
        // 创建 tech 分组结构
        const techGroups = Array.from(
          d3.group(
            this.techHierarchy,
            d => d.dim,
            d => d.parent
          ),
          ([dim, parentGroups]) => ({
            key: dim,
            values: Array.from(parentGroups, ([parent, techs]) => ({
              key: parent,
              values: Array.from(techs)
            }))
          })
        );
        
        // 计算列宽
        const headerHeight = height / 60 * 15;
        const cellWidth = innerWidth / this.techHierarchy.length;
        const cellHeight = height / 85;
        
        // 绘制表头
        let xPosition = 0;
        techGroups.forEach(dimGroup => {
          const dimName = dimGroup.key;
          const parentGroups = dimGroup.values;
          
          // 绘制 dim 分组标题
          g.append('rect')
            .attr('x', xPosition)
            .attr('y', 0)
            .attr('width', parentGroups.reduce((sum, pg) => sum + pg.values.length, 0) * cellWidth)
            .attr('height', headerHeight / 10)
            .attr('fill', '#ffffff')
            .attr('stroke', '#ccc')
            .attr('stroke-width', 1);
          
          g.append('text')
            .attr('x', xPosition + (parentGroups.reduce((sum, pg) => sum + pg.values.length, 0) * cellWidth) / 2)
            .attr('y', headerHeight / 20)
            .attr('text-anchor', 'middle')
            .attr('dominant-baseline', 'middle')
            .text(dimName)
            .style('font-weight', 'bold')
            .style('font-size', '10px');
          
          // 绘制 parent 分组标题
          parentGroups.forEach(parentGroup => {
            const parentName = parentGroup.key;
            const techsInGroup = parentGroup.values;
            
            g.append('rect')
              .attr('x', xPosition)
              .attr('y', headerHeight / 10)
              .attr('width', techsInGroup.length * cellWidth)
              .attr('height', headerHeight / 10)
              .attr('fill', '#ffffff')
              .attr('stroke', '#ccc')
              .attr('stroke-width', 1);
            
            g.append('text')
              .attr('x', xPosition + (techsInGroup.length * cellWidth) / 2)
              .attr('y', headerHeight / 20 * 3)
              .attr('text-anchor', 'middle')
              .attr('dominant-baseline', 'middle')
              .text(parentName)
              .style('font-size', '6px');
            
            // 绘制 tech 名称
            g.append('rect')
              .attr('x', xPosition)
              .attr('y', headerHeight / 10 * 2)
              .attr('width', techsInGroup.length * cellWidth)
              .attr('height', headerHeight / 5 * 3)
              .attr('fill', '#ffffff')
              .attr('stroke', '#ccc')
              .attr('stroke-width', 1);
            
            techsInGroup.forEach(tech => {
              const textX = xPosition + cellWidth / 2;
              const textY = headerHeight / 5 + headerHeight / 5 * 3 / 2;
              
              g.append('text')
                .attr('x', textX)
                .attr('y', textY)
                .attr('text-anchor', 'middle')
                .attr('dominant-baseline', 'middle')
                .attr('transform', `rotate(-90, ${textX}, ${textY})`)
                .style('font-size', '6px')
                .text(tech.tech);
              
              xPosition += cellWidth;
            });
          });
        });
        
        // 绘制 count 行
        let countY = headerHeight / 5 * 4;
        this.techHierarchy.forEach((tech, index) => {
          const xPosition = index * cellWidth;
          
          g.append('rect')
            .attr('x', xPosition)
            .attr('y', countY)
            .attr('width', cellWidth)
            .attr('height', cellHeight)
            .attr('fill', '#f0f0f0')
            .attr('stroke', '#ccc')
            .attr('stroke-width', 1);
          
          g.append('text')
            .attr('x', xPosition + cellWidth / 2)
            .attr('y', countY + cellHeight / 2)
            .attr('text-anchor', 'middle')
            .attr('dominant-baseline', 'middle')
            .style('font-size', '6px')
            .text(tech.count);
        });
        
        // 绘制项目行
        this.projects.forEach((project, i) => {
          const yPosition = headerHeight / 5 * 4 + (i + 0.5) * cellHeight + cellHeight * 1.5;

          // 绘制左侧 project name 的背景框
          g.append('rect')
            .attr('x', -this.margin.left) // 左侧对齐
            .attr('y', yPosition - cellHeight / 2) // 调整到项目名称的中心
            .attr('width', this.margin.left-5) // 宽度为左侧边距
            .attr('height', cellHeight) // 高度与单元格一致
            .attr('fill', '#ffffff') // 背景颜色
            .attr('stroke', '#ccc') // 边框颜色
            .attr('stroke-width', 1);

          // 绘制项目名称
          g.append('text')
            .attr('x', -10)
            .attr('y', headerHeight / 5 * 4 + (i + 0.5) * cellHeight + cellHeight * 1.5)
            .attr('text-anchor', 'end')
            .attr('dominant-baseline', 'middle')
            .style('font-weight', 'bold')
            .style('font-size', '5.5px')
            .text(project.name)
            .style('cursor', 'pointer') // 添加鼠标样式
            .on('mouseover', () => {
              // 高亮当前行的有颜色的格子
              g.selectAll('rect')
                .attr('opacity', function () {
                  const rect = d3.select(this);
                  const fillColor = rect.attr('fill');
                  const yPosition = +rect.attr('y');
                  const isCurrentRow = yPosition === headerHeight / 5 * 4 + i * cellHeight + cellHeight * 1.5;

                  // 当前行的有颜色格子保持不变，其他格子降低透明度
                  return isCurrentRow && fillColor !== '#ffffff' ? 1 : 0.2;
                });
                // 降低其他项目名称的透明度
                g.selectAll('text')
                  .attr('opacity', function () {
                    const text = d3.select(this);
                    const textYPosition = +text.attr('y');
                    const isCurrentProject = textYPosition === headerHeight / 5 * 4 + (i + 0.5) * cellHeight + cellHeight * 1.5;

                    // 当前项目名称保持不变，其他项目名称降低透明度
                    return isCurrentProject ? 1 : 0.2;
                  });
              })
              .on('mouseout', () => {
                // 恢复所有格子的透明度
                g.selectAll('rect').attr('opacity', 1);
                // 恢复所有项目名称的透明度
                g.selectAll('text').attr('opacity', 1);
              })
              .on('click', () => {
                this.handleCellClick(project.name);
              });
          
          // 绘制单元格
          let xPosition = 0;
          this.techHierarchy.forEach(tech => {
            // 格式化 tech.tech 以匹配 project.techUsage 的键名
            const formattedTech = tech.tech.replace(/\s+/g, ' ').toLowerCase();
            const value = project.techUsage[formattedTech] || 0;
            
            // 根据技术名称分配颜色
            const color = this.getColorForTech(formattedTech);
            
            const cellColor = value > 0 ? color : '#ffffff';
            
            g.append('rect')
              .attr('x', xPosition)
              .attr('y', headerHeight / 5 * 4 + i * cellHeight + cellHeight * 1.5)
              .attr('width', cellWidth)
              .attr('height', cellHeight)
              .attr('fill', cellColor)
              .attr('stroke', '#ccc')
              .attr('stroke-width', 1)
              .on('click', () => this.handleCellClick(project.name))
              .on('mouseover', (event) => {
                // 仅当格子有颜色时触发
                if (cellColor !== '#ffffff') {
                  d3.select(event.currentTarget).attr('opacity', 0.8);

                  // 获取技术信息
                  const techInfo = this.techMap.get(tech.tech);
                  const [x, y] = d3.pointer(event, this.$refs.matrixTable);

                  // 创建 tooltip 容器
                  const tooltip = g.append('foreignObject')
                    .attr('class', 'tooltip')
                    .attr('x', x - 350) // 鼠标右侧偏移 10px
                    .attr('y', y + 10) // 鼠标下方偏移 10px
                    .attr('width', 150) // 设置最大宽度
                    .attr('height', 100) // 设置最大高度
                    .style('overflow', 'visible');

                  // 在 foreignObject 中添加 HTML 内容
                  tooltip.append('xhtml:div')
                    .style('background', 'rgba(255, 255, 255, 0.9)')
                    .style('border', '1px solid #ccc')
                    .style('border-radius', '4px')
                    .style('padding', '5px')
                    .style('font-size', '8px')
                    .style('color', 'black')
                    .style('max-width', '150px') // 限制最大宽度
                    .style('word-wrap', 'break-word') // 启用自动换行
                    .text(`${project.name}: ${tech.tech} - ${techInfo.subparent} (${techInfo.count})`);
                }
              })
              .on('mouseout', (event) => {
                d3.select(event.currentTarget).attr('opacity', 1);
                g.selectAll('.tooltip').remove();
              })
            
            xPosition += cellWidth;
          });
        });
        
      } catch (error) {
        console.error('Error drawing taxo matrix:', error);
      }
    }
  },
  mounted() {
    this.drawTaxoMatrix();
  }
};
</script>

<style scoped>
div {
  overflow: auto;
}
</style>