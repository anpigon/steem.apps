<template>
  <div  class="container">
    <div class="row"><br /></div>
    <div class="row">
      Mobile Web OCR with Tesseract.js      
    </div>
    <div class="row"><br /></div>
  	<div class="row">
  		<div class="col-md-12">
        <div class="input-group" id="adv-search">
            <input id="fileName" type="text" class="form-control width_70" placeholder="Tesseract.js" readOnly />
            <select class="form-control width_30" id="sel_lang">
              <option value="eng">eng</option>
              <option value="kor">kor</option>
              <option value="jpn">jpn</option>
              <option value="fra">fra</option>
            </select>
            <div class="input-group-btn ">
                <div class="btn-group" role="group">
                  <button type="button" class="btn btn-success" v-on:click="capture1_click"><span class="glyphicon glyphicon-camera" aria-hidden="true"></span></button>                          
                  <button type="button" class="btn btn-success" v-on:click="capture2_click"><span class="glyphicon glyphicon-picture" aria-hidden="true"></span></button>
                </div>
            </div>
        </div>
      </div>
    </div><!-- end row. -->
    <div class="row"><br /></div>
    <div class="row">
      <input class="hidden" id="capture1" type="file" accept="image/*" capture>
      <input class="hidden" id="capture2" type="file" accept="image/*">
      <div class="col-md-12 text-center">
        <img id="imgSrc" src="" class="" />
        <br /><br />
        <div class="panel panel-success">
          <div class="panel-heading">
            <h5 class="panel-title">OCR Result</h5>
          </div>
          <div class="panel-body" id="textDiv">{{ data.ocrResult}}</div>
        </div>        
      </div>
    </div>
  </div><!-- end container -->
</template>
<script>
var data = {
  ocrResult : ""
};
module.exports = {
  name: 'search',
  data: function () {
    return {
      data: data
    }
  },
  computed: {
    hasResult: function () {
      return this.posts.length > 0
    }
  }
  ,methods: {
    searchTerm: function () {
      // using JSONPlaceholder
      var baseURI = 'https://www.google.co.kr/search?newwindow=1&ei=iKgGWuPSB4nq8AXVyryYBg&q=site%3Asteemit.com+%EC%8B%AC%EC%8B%AC%ED%92%80%EC%9D%B4&oq=site%3A&gs_l=mobile-gws-serp.3.2.35i39k1l3j0l2.1976.6230.0.8334.13.10.2.0.0.0.134.994.1j8.9.0....0...1.1.64.mobile-gws-serp..3.8.686.3..0i131k1.91.6K4jmL9FzBg.js';
      this.$http.get(`${baseURI}`)
      .then((result) => {
        console.log(result)
        this.posts = result.data
      })
    }
    , capture1_click : function(){ capture1_click(); }
    , capture2_click : function(){ capture2_click(); }
  }
  ,created: function() {
    $.when(
        $.getScript( "https://cdn.jsdelivr.net/gh/naptha/tesseract.js@v1.0.14/dist/tesseract.min.js" ),
        //$.getScript( "/mypath/myscript2.js" ),
        
        $.Deferred(function( deferred ){
            $( deferred.resolve );
        })
    ).done(function(){

        alert("loading end");

    });
		$.getScript();		
	}
  , mounted:function () {
    var input = document.querySelector('#capture1');
    $("input[type=file]").on('change', function () {      
      var file = this.files[0];
      $("#fileName").val(file.name);
      var sel_lang = $("#sel_lang").val();
      //console.log("input.onchange", file, sel_lang);
      displayAsImage(file, $("#imgSrc"));
      
      waitingDialog.show('Processing Tesseract', { progressType: 'success' });
      data.ocrResult = "";
      Tesseract.recognize(file, {
          lang: sel_lang,
          tessedit_char_blacklist: ''
      })
      .progress(function(message){ 
        //console.log(message, parseInt(message.progress * 100)); 
        waitingDialog.setMessage(message.status+"......");
        waitingDialog.setProgressWidth( parseInt(message.progress * 100) ) 
      })
      .catch(function(err){ 
        console.error(err); 
        alert(err);
      })
      .then(function(result){
          console.log(result);
          data.ocrResult = result.text;
          waitingDialog.hide();
      })
      .finally( function(resultOrError){ waitingDialog.hide(); console.error(resultOrError); })


    });
  }
}

function displayAsImage(file, img) {
  var imgURL = URL.createObjectURL(file);
  img.on("load", function() {
    URL.revokeObjectURL(imgURL);
  });
  img.attr("src", imgURL);
}
function capture1_click(){
  $("#capture1").click();
}
function capture2_click(){
  $("#capture2").click();
}
</script>
<style>

img{
  max-width: 100%;
}
.width_70{
  width:65% !important;
}
.width_30{
  width:35% !important;
}

body {
    padding-top: 50px;
}
.dropdown.dropdown-lg .dropdown-menu {
    margin-top: -1px;
    padding: 6px 20px;
}
.input-group-btn .btn-group {
    display: flex !important;
}
.btn-group .btn {
    border-radius: 0;
    margin-left: -1px;
}
.btn-group .btn:last-child {
    border-top-right-radius: 4px;
    border-bottom-right-radius: 4px;
}
.btn-group .form-horizontal .btn[type="submit"] {
  border-top-left-radius: 4px;
  border-bottom-left-radius: 4px;
}
.form-horizontal .form-group {
    margin-left: 0;
    margin-right: 0;
}
.form-group .form-control:last-child {
    border-top-left-radius: 4px;
    border-bottom-left-radius: 4px;
}

@media screen and (min-width: 768px) {
    #adv-search {
        width: 500px;
        margin: 0 auto;
    }
    .dropdown.dropdown-lg {
        position: static !important;
    }
    .dropdown.dropdown-lg .dropdown-menu {
        min-width: 500px;
    }
}
</style>
