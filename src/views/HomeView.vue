<template>
  <meta name="viewport" content="initial-scale=1,maximum-scale=3,minimum-scale=1" />
<div style="background-color: #f5f5f5;">
  <section class="homepage" id="home" >
      <div class="content">
        <div class="text">
          <h1>{{Page1.title}}</h1>
          <p >{{Page1.content}}</p>
        </div>

        <!-- <a href="#Articles">{{details0 }} >></a> -->
        <a @click="scrollToSection('Articles')">{{details0 }} >></a>
      </div>
    </section>

    <!-- <section class="Aboards" id="Aboards">
      
      <table class="AboardsTable">
        <tr>
         <td colspan="3"> <h2>{{Page2.title[Language]}}</h2> </td> 
        </tr>
      
      <tr v-for="item in Page2.content">
        <td style="text-align: center;color:rgb(164, 162, 162);">{{ item.time }}</td>
        <td style="text-align: left;">{{ item.name[Language] }}</td>
        <td style="text-align: center;color:rgb(164, 162, 162);">more >></td>
      </tr>
    </table> 
  </section> -->

  <div style="height: 60px;"></div>

  <section class="Articles" id="Articles">
      <!-- <h2>Introduction of emotional geography</h2> -->
      <div class="showPages" >
        <div class="imageRight">
            <img class="imageShow" style="width: 50%"  :src="require('@/assets/images/first/smileearth2.png')" alt="图片描述">
          </div>
          <div class="textLeft">
            <!-- <h1>{{Page2.title[Language]}}</h1> -->
            <p>
              Affective visualization design is an emerging research direction focused on communicating and influencing emotion through visualization. 
            However, as revealed by previous research, this area is highly interdisciplinary and involves theories and practices from diverse fields and disciplines, thus awaiting analysis from more fine-grained angles. 
            To address this need, this work focuses on a pioneering and relatively mature sub-area, affective geovisualization design, to further the research in this direction and provide more domain-specific insights. 
            A corpus was collected and analyzed, containing 62 affective geovisualization design practices.
              <!-- In 2001, Anderson and Smith introduced the concept of emotional geographies, emphasizing the key role of emotion in studying how humans interact with places and spaces. 
              Since then, a series of conferences, journal issues, and books focused on emotional geography have emerged, establishing this area as a significant substream within geographical research. 
              According to the summarization by Bondi, emotional geography was grounded in a set of geographical traditions that were deeply influenced by philosophies and methodologies from the humanities and social sciences, such as humanistic geography, feminist geography, and non-representational geography. 
              Rather than focusing on the scientific laws governing the Earth and its atmosphere (e.g., climate, hydrology, soil), these research threads exhibit strong attention to human and their subjective experience with places. 
              For example, humanistic geography sought to understand the subjective and experienced “life-world”, seeing places as a center of meaning constructed by personal experience.  -->
              <!-- Under this epistemology, researchers have found that the same location can evoke different “senses of place” among individuals, influenced by personal histories and socio-cultural contexts (e.g., residential status, age stage). 
              Even the same geographic scale (e.g., the distance between work and home) can be perceived differently given different moods and states (e.g., the road feels lengthy and tiring after a day of overtime work, but becomes shorter and more pleasing on the last day before a holiday). 
              Sometimes, individuals may even exhibit extreme emotions to places, such as agoraphobia (i.e., an intense fear of open or crowded spaces) and madness.  -->
            </p>
            <div class="buttonRight">
              <a href="/DataSet">{{details }} >></a>
            </div>
          </div>
      </div>
  </section>
  
 

  <div style="height: 30px;"></div>

  <section class="Articles" id="Articles">
      <!-- <h2>Introduction of our work</h2> -->
    <div class="showPages" >
      <div class="imageRight">
          <img class="imageShow" :src="require('@/assets/images/first/PPP.svg')" alt="图片描述">
        </div>
        <div class="textRight">
          <!-- <h1>{{Page2.title[Language]}}</h1> -->
          <p>
            Through an analysis of a curated corpus of affective geovisualization designs using the Person-Process-Place (PPP) model from geographic theory, we derived a design taxonomy that characterizes a variety of methods for eliciting and enhancing emotions through geographic visualization. 
            We also identified four underlying high-level design paradigms of affective geovisualization design (e.g., computational, anthropomorphic) that guide distinct approaches to linking geographic information with human experience. 
          </p>
          <div class="buttonRight">
    <a href="/Taxonomy">{{details2 }} >></a>
  </div>
        </div>
      </div>
  </section>
<section></section>


  <!-- <section></section> -->
   <!-- <section class="Articles" >
    <h2  >11111</h2>
    <div class="showPages" >
        <div class="imageRight">
          <img class="imageShow" :src="Page2.img" alt="图片描述">
        </div>
        <div class="textLeft">
          <h1>{{Page2.title}}</h1>
          <p>{{Page2.content}}</p>
        </div>
        
      </div>
    </section> -->
  <!-- <section></section> -->
  
 <!-- <section class="homepage"  :style="{ backgroundImage: `url(${Page3.img})` }" >
      <div class="content">
        <div class="text">
          <h1>{{Page3.title[Language]}}</h1>
          <p>{{Page3.content[Language]}}</p>
        </div>
        <a href="/service">{{ details[Language] }} >></a>
      </div>
    </section>

   
    <section class="homepage"  :style="{ backgroundImage: `url(${Page6.img})` }" >
      <div class="content">
        <div class="text">
          <h1>{{Page6.title[Language]}}</h1>
          <p>{{Page6.content[Language]}}</p>
        </div>
        <a href="/Connection">{{ details[Language] }} >></a>
      </div>
    </section>
    
    <section class="homepage"  :style="{ backgroundImage: `url(${Page5.img})` }" >
      <div class="content">
        <div class="text">
          <h1>{{Page5.title[Language]}}</h1>
          <p>{{Page5.content[Language]}}</p>
        </div>
        <a href="/Report">{{ details[Language] }} >></a>
      </div>
    </section> -->

  </div>
</template>

<script>
import "@/assets/css/style.css";
export default {
  watch:{
      '$store.state.Language':{
        handler(newVal){
          this.Language=newVal;
        },
        deep: true
      }
    },
    created(){
        // axios.get('Aboard/list').then(res=>{
        //     this.data = res.data;
      
        // }).catch(error => {
      //     console.error('Failed :', error);
      //   });

    },
  data() {
    let self = this;
    return {
      Language:this.$store.state.Language,
      details0:"Explore",
      details:"Explore our corpus",
      details2:"Explore the design taxonomy",
      Page1:{title:"Mapping What I Feel",content:"Understanding Affective Geovisualization Design Through the Lens of People-Place Relationships",img:require('@/assets/images/first/home1.png')},
      Page2:{title:["Aboard","Aboard"],content:['11','EN'],img:require('@/assets/images/first/home1.png')},
      Page3:{title:["What We Do","What We Do"],content:['11','111'],img:require('@/assets/images/first/home1.png')},
      Page4:{title:["About","About"],content:['11','EN'],img:require('@/assets/images/first/home1.png')},
      Page5:{title:["Insights","Insights"],content:['11','11'],img:require('@/assets/images/first/home1.png')},
      Page6:{title:["Our Connections","Our Connections"],content:['11','11'],img:require('@/assets/images/first/home1.png')}

     
    };
  },
  methods: {
  scrollToSection(sectionId) {
    const section = document.getElementById(sectionId);
    if (section) {
      section.scrollIntoView({ behavior: 'smooth' }); // 平滑滚动到目标部分
    }
  }
},
 
};
</script>

<style>
#home{
  background-image:  linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.5)),url('@/assets/images/first/home2.jpg');
}
@media screen and (max-width: 860px) {
  #home{
    background-image: url('@/assets/images/first/home2.jpg');
  }
}
.AboardsTable{
  background-color: #fff;
  box-shadow: 10px 10px 5px #888888;
  width:80%;
  border-spacing: 20px;
  border: 1px solid rgba(197, 203, 208, 0.773);
}
td, th {
  border: none; /* 隐藏单元格内边框 */
  padding: 1px; /* 可选，设置单元格内边距 */
}

/* .linkContainer {
  display: flex; 
  justify-content: space-between;
  gap: 10px;
} */




.Articles {
  padding: 20px; /* 四周内边距 */
  /* background-color: rgba(233, 230, 226, 1); 
  border-radius: 10px; 
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1), 0px -4px 8px rgba(0, 0, 0, 0.1), 
              4px 0px 8px rgba(0, 0, 0, 0.1), -4px 0px 8px rgba(0, 0, 0, 0.1);  */
  padding-bottom: 0px;
}



.Articles a {
  text-decoration: none;
  font-size: 18px;
  border: 1px solid #fff;
  border-radius: 5px;
  padding: 10px 10px;
  color: #3c3838;
  background-color: #ffffff1d;
  transition: 0.5s;
}

.Articles a:hover {
  background-color: #3c38386d;
  color: #fff;
}



.showPages {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  min-height: 50%;
  /* background-color: rgba(233, 230, 226, 0.5);  */
  /* border-radius: 10px; 设置内容区域的圆角 */
  /* box-shadow: 0 0 10px rgba(0, 0, 0, 0.2); 设置内容区域的阴影效果 */
  padding: 20px;
  color:black;
  gap: 20px; /* 添加间距 */
}

.buttonRight {
  display: flex;
  /* justify-content: flex-end;  */
  /* 将按钮推到容器最右边 */
  margin-top: 10px; /* 按钮与文字之间的间距 */
  padding-right: 40px;
  padding-bottom: 20px;
}

.buttonRight a {
  text-decoration: none;
  font-size: 18px;
  border: 1.5px solid #fff;
  border-radius: 10px;
  padding: 10px 10px;
  color: #fff;
  background-color: #11334A;
  transition: 0.5s;
  white-space: nowrap; /* 防止按钮文字换行 */
}

.buttonRight a:hover {
  background-color: #11334adf;
  color: #fff;
}

/* 文字部分样式 */
.textLeft {
  /* margin-right: 10px;  */
  text-align: justify; 
  font-size: 16px; /* 调整文字大小 */
  line-height: 2; /* 行高 */
  color: #333; /* 文字颜色 */
  width: 60%;
  letter-spacing: 0.05em; 
}

.textRight {
  /* margin-right: 10px;  */
  text-align: justify; 
  font-size: 16px; /* 调整文字大小 */
  line-height: 2; /* 行高 */
  color: #333; /* 文字颜色 */
  width: 60%;
  letter-spacing: 0.05em;
}

/* 图片部分样式 */
.imageRight {
  flex: 1; /* 占据剩余空间 */
  text-align: center; /* 图片居中 */
}

.imageLeft {
  float:left; 
  margin: 5px;
  height: 100%;
  flex: 1;
  text-align: center;
}

.imageShow {
  max-width: 70%; /* 图片宽度自适应 */
  height: auto; /* 保持比例 */
  border-radius: 8px; /* 图片圆角 */
}



</style>
