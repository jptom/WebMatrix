<script>
  import { browser } from '$app/environment';
  let n_words = 0;
  let n_lines = 0;
  let space_symbol = "&nbsp;";
  let line_texts = Array();
  let remaining_lengths = Array();
  let length = Array();
  let heads = Array();
  let html_lines = "";
  const oPTag = '<p class="vertical">';
  const cPTag = '</p>';
  const oHeadTag = '<span class="head">';
  const cHeadTag = '</span>'
  let height, width;
  let n;
  let index;
  let copy_element;
  let original_index;
  let x, y;
  let is_run=false;
  let id = 0;
  let change = false;
  let speed=40;
  let hidden = false;
  let asynchronous = false;
  let gradation = false;
  let pause = false;
  const alphabet = "abcdefghijklmnopqrstuvwxyz"
  const numbers = "1234567890"
  const Alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
  const symbols = "!@#$%^&*+"
  const candidate_words = alphabet + Alphabet + numbers + symbols; 

  if (browser) {
    let root = document.documentElement;
    
    console.log(window.innerWidth, window.innerHeight);
    width = window.innerWidth;
    height = window.innerHeight;
    let fontSize = window.getComputedStyle(root).getPropertyValue('font-size').slice(0, -2);
    console.log(fontSize)
    n_words = Math.floor(height / fontSize);
    n_lines = Math.floor(width / fontSize);
    console.log(n_words, n_lines)
  }

  function clear_screen (){
    console.log("clear_screen")
    line_texts = Array(n_lines).fill(null).map(()=>Array(n_words).fill(' '));
    length = Array(n_lines).fill(0);
    heads = Array(n_lines).fill(null).map(()=>Array(n_words).fill(0));
    remaining_lengths =  Array(n_lines).fill(null).map(()=>Math.floor(Math.random()*(n_words-1))+2);
    console.log(remaining_lengths)
    const MIN = Math.min(...remaining_lengths);
    console.log(MIN)
    remaining_lengths = remaining_lengths.map(element => element - MIN + 1);
    console.log(remaining_lengths)
  }

  function html_rendering(){
    html_lines = line_texts.map((line, x) =>
      oPTag + line.map((char, y)=>{
        if(heads[x][y] == 0){
          return space_symbol;
        }
        else if(heads[x][y]==1){
          return oHeadTag+char+cHeadTag;
        }
        else if(gradation){
          return `<span style=opacity:${heads[x][y]}>`+line[y]+'</span>';
        }
        else{
          return char;
        }
      }).join('') + cPTag
    ).join('');
  }

  function move(){
    for(x=0; x<n_lines; x++){
      if(asynchronous & Math.random()<0.5){
        continue;
      }
      for(y=n_words-1; y>0; y--){
        if(line_texts[x][y-1]==' '){
          line_texts[x][y]=' ';
        }          
        else if(line_texts[x][y]==' '){
          line_texts[x][y] = candidate_words[Math.floor(Math.random() * candidate_words.length)];
        }
        else if(change & Math.random()<0.05){
          line_texts[x][y] = candidate_words[Math.floor(Math.random() * candidate_words.length)];
        }
        heads[x][y] = heads[x][y-1];
      }

      remaining_lengths[x]--;
      if(remaining_lengths[x]==0){
        remaining_lengths[x] = line_texts[x][0]==' '? Math.floor(Math.random()*(n_words-3))+4 : Math.floor(Math.random()*(n_words-1))+2;
        if(line_texts[x][0]==' '){
          line_texts[x][0] = candidate_words[Math.floor(Math.random() * candidate_words.length)];
          heads[x][0] = 1;
          length[x] = remaining_lengths[x];
        }
        else{
          line_texts[x][0] = ' ';
          heads[x][0] = 0;
          length[x] = 0;
        }
      }
      else{
        heads[x][0] = length[x]? (remaining_lengths[x]+0.9)/length[x] : 0;
      }
    }
    

  }
  function update(){
    //console.log("update")
    //console.log(heads)
    if(!pause){
      move();
      html_rendering();
    }

    setTimeout(update, speed);

    //console.log(line_texts)
  }
  function onKeyDown(event){
    switch(event.key){
      case '1':
      case '2':
      case '3':
      case '4':
      case '5':
      case '6':
      case '7':
      case '8':
      case '9':
        speed = Number(event.key)*10;
      case 'a':
        asynchronous = !asynchronous;
        break;
      case 'd':
        change = false;
        asynchronous = false;
        gradation = false;
        speed = 40;
        break;
      case 'g':
        gradation = !gradation
        break;
      case 'k':
        change = !change;
        break;
      case 'h':
        hidden = !hidden;
        break;
      case 'p':
        pause = !pause;
        break;
      case 'r':
        clear_screen();
        break;
      }
      html_rendering();
    }
  /*-- main --*/
  
  clear_screen();
  html_rendering();
  update();


</script>

<svelte:head>
  <title>Matrix</title>
	<meta name="description" content="Matrix written in SvelteKit" />
</svelte:head>

<dev class="screen vertical">
  <!--<span class="vertical">matrix</span>-->
  {@html html_lines}
</dev>
<dev class="description" class:hidden>
  <h1>Web-Matrix</h1>
  <h2>KEYBOARD CONTROL:</h2>
  <p>1~9 : scrolling speed</p>
  <p>d : Apply default setting.</p>
  <p>g : Gradation scrolling.</p>
  <p>h : Show or hide descripton.</p>
  <p>k : Characters change while scrolling.</p>
  <p>p : Pause</p>
  <p>r : Restart scrolling.</p>
</dev>
<svelte:window on:keydown|preventDefault={onKeyDown} />
<style>
  
  .screen {
    writing-mode: vertical-rl;
    text-orientation: upright;
    background-color: rgb(0, 0, 0);
  }
  .screen > :global(.vertical) {
    color: rgb(0, 255, 0);
    margin: 0%;
  }
  .screen > :global(.vertical) > :global(.head) {
    color: rgb(255, 255, 255);
  }
  .description{
    visibility: visible;
    position: absolute;
    border: 5px solid #ffffff;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    margin: 10% auto auto auto;
    border-radius: 10px; 
    width:500px;
    height:500px;
    background-color: rgb(250, 150, 50);
  }
  .description.hidden{
    visibility: hidden;
  }
  h1 {
    text-align: center;
  }
</style>

