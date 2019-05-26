# Flying-Feet

<html>
<head>
	<link rel="stylesheet" href="Main.css">
	<link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
	<link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
	<script src="https://www.gstatic.com/firebasejs/5.7.3/firebase.js"></script>
	<script src="https://apis.google.com/js/platform.js" async defer></script>
	<script src="https://www.gstatic.com/charts/loader.js" type="text/javascript"></script>
</head>
<Body>
	<div class="content">
		<div class="pill-nav">
			<a href="home.html">Home</a>
			<a class="active" href="catalog.html">Catalog</a>
			<a href="about.html">About</a>
		</div>
		<div class="w3-container w3-center w3-animate-top">
			<h1 style="strong; text-align: center;">Welcome to Flyingfeet</h1>
		</div>
		<div class="w3-container w3-center w3-animate-left">
			<h2 style="text-align: center">The number one place to browse shoes</h2>
		</div>
		<div id="divLoading">
			<br /><br />
			Please wait...
		</div>
		<div id="divLogin" class="hidden">
			<hr />
			<br /><br />
			Hello!
			<br />
			Click 'Sign In' to login with your google account.<br />
			<button id="btnSignIn" style="background-color: aqua; padding: 5px;" onclick="signin()">Sign in</button>
		</div>
		<div id="divTweet" class="hidden">
			<hr />
			<div class="title">
				<div id="myphoto" style="display: inline-block;"></div>
				<div id="mydata" style="display: inline-block; vertical-align: top; padding-top: 5px; padding-left: 5px;"></div>
				<div style="display: inline-block; vertical-align: top; padding-left: 20px;">
					<button id="btnSignOut" style="background-color: aqua; padding: 5px;" onclick="signout()">Sign out</button>
				</div>
			</div>
			<div id="error" style="color: red;"></div>
			<table class="tweet">
				<tr>
					<td>Product Name <font style="color: red;">*</font>:</td>
					<td><input id="twit" type="text" name="thetweet"></td>
				</tr>
				<tr>
					<td>Contact Information <font style="color: red;">*</font>:</td>
					<td><input id="twitInfo" type="text" name="thetweet"></td>
				</tr>
				<tr>
					<td>Select an Image<font style="color: red;"> *</font>:</td>
					<td><input id="twitFile" type="file" onchange="encodeImageFileAsURL();" /><div id="twitImg" class="hidden"></div></td>
				</tr>
				<tr>
					<td>Select a Size:</td>
					<td>
						<select id="twitSize">
							<option value="">Not Selected</option>
							<option value="8">8</option>
							<option value="9">9</option>
							<option value="10">10</option>
							<option value="11">11</option>
							<option value="12">12</option>
							<option value="13">13</option>
						</select>
					</td>
				</tr>
				<tr>
					<br>
					<td>Select a Color:</td>
					<td>
						<select id="twitColor">
							<option value="">Not Selected</option>
							<option value="red">Red</option>
							<option value="blue">Blue</option>
							<option value="black">Black</option>
							<option value="gray">Gray</option>
						</select>
					</td>
				</tr>
				<tr>
					<td></td>
					<td>
						<br>
						<button type="button" style="background-color: aqua; padding: 5px" onclick="tweet()">Submit</button>
					</td>
				</tr>
			</table>
			<br />
			<hr />
			<div id="mytweets"></div>
			<div id="likeschart"></div>
		</div>
	</div>
	<script>
		// Initialize Firebase
		google.charts.load("current", { packages: ['corechart'] });

		var config = {
			apiKey: "AIzaSyAoeJweLWYqEbT3e8-TBDru_diAZrzhYBo",
			authDomain: "test-dcfe4.firebaseapp.com",
			databaseURL: "https://test-dcfe4.firebaseio.com",
			projectId: "test-dcfe4",
			storageBucket: "test-dcfe4.appspot.com",
			messagingSenderId: "169545986094"
		};

		firebase.initializeApp(config);

		// Check to see if you are logged in
		firebase.auth().onAuthStateChanged(function (user) {
			if (user == null) {
				document.getElementById("divLoading").classList.add("hidden");
				document.getElementById("divLogin").classList.remove("hidden");
				document.getElementById("divTweet").classList.add("hidden");
				return;
			} else {
				document.getElementById("divLoading").classList.add("hidden");
				document.getElementById("divLogin").classList.add("hidden");
				document.getElementById("divTweet").classList.remove("hidden");

				userId = user.uid;
				name = user.displayName;
				imageUrl = user.photoURL;
				email = user.email;

				// write user data to users
				writeUserData(userId, name, email, imageUrl);

				// write data to document
				mydiv = document.getElementById("mydata");
				mydiv.innerHTML = name
				myphotodiv = document.getElementById("myphoto");
				myphotodiv.innerHTML = "<img class='myphoto' src='" + imageUrl + "'/>";

				firebase.database().ref('/tweets/' + userId).once('value').then(function (snapshot) {
					var data = (snapshot.val());
					var keys = [];

					if (data == null) {
						console.log("No data found at /tweets");
					} else {
						snapshot.forEach(function (childSnapshot) {
							keys.push(childSnapshot.key);
						});

						updatetweets(keys, data);
					}
				});
			} // end user null check
		}); // end check auth state

		// write user data
		function writeUserData(userId, name, email, imageUrl) {
			firebase.database().ref('users/' + userId).set({
				username: name,
				email: email,
				profile_picture: imageUrl
			});
		}

		function updatetweets(keys, data) {
			var tweets = "<table class='products'>";
			var i = 0;

			var chart = new google.visualization.DataTable();
			chart.addColumn('string', 'Product');
			chart.addColumn('number', 'Likes');
   
			for (var u in data) {
				var d = new Date(data[u].time);
				console.log(data[u]);

				tweets = tweets + "<tr>";
				tweets = tweets + "<td class='first-col'>";
				tweets = tweets + "		<img src='" + data[u].img + "' width='100px' height='100px'>";
				tweets = tweets + "		<div><div id='like" + i.toString() + "'>" + (data[u].likesCount == undefined || isNaN(data[u].likesCount) ? "0" : data[u].likesCount) + "</div>&nbsp;like(s)</div>";
                tweets = tweets + "		<div><div id='dislike" + i.toString() + "'>" + (data[u].dislikesCount == undefined || isNaN(data[u].dislikesCount) ? "0" : data[u].dislikesCount) + "</div>&nbsp;dislike(s)</div>";
				tweets = tweets + "	<a href='#' onClick=\"like('" + keys[i] + "', " + i.toString() + "); return false;\" ><img src='thumbsup.jpg' style='height: 25px; width: 25px;'/></a>";
				tweets = tweets + "	<a href='#' onClick=\"dislike('" + keys[i] + "', " + i.toString() + "); return false;\" ><img src='thumbsdown.jpg' style='height: 25px; width: 25px;'/></a>";
				tweets = tweets + "</td>";
				tweets = tweets + "<td class='second-col'>";
				tweets = tweets + "Product Name: " + data[u].tweet + "<br />" + "<br />";
				tweets = tweets + "Contact Information: " + data[u].info + "<br />" + "<br />";
				tweets = tweets + "Color: " + data[u].color;
				tweets = tweets + "</td>";
				tweets = tweets + "</tr>";

				chart.addRows([[data[u].tweet, Number((data[u].likesCount == undefined ? "0" : data[u].likesCount))]]);

				i++;
			}
			tweets = tweets + "</table>";

			var mytdiv = document.getElementById("mytweets");
			mytdiv.innerHTML = tweets;

			console.log(chart);

			// Set chart options
			var options = {
				'title': 'Likes Chart',
				'width': 600,
				'height': 300
			};
            


			// Instantiate and draw our chart, passing in some options.
			var likesChart = new google.visualization.PieChart(document.getElementById('likeschart'));
			likesChart.draw(chart, options);
            
		}

		// write tweets to firebase
		function tweet() {

			var twit = document.getElementById("twit");
			var twitInfo = document.getElementById("twitInfo");
			var twitImg = document.getElementById("twitImg");
			var twitColor = document.getElementById("twitColor");
			var twitSize = document.getElementById("twitSize");
			var error = document.getElementById("error");
			var js_time = Date.now();

			if (twit.value.length == 0) {
				error.innerHTML = "Please Enter a Valid Product Name";
				return;
			}

			if (twitInfo.value.length == 0) {
				error.innerHTML = "Please Enter a Valid Contact Information field";
				return;
			}

			if (twitImg.innerHTML == "") {
				error.innerHTML = "Please Select a Valid Image";
				return;
			}

			var tweetid = firebase.database().ref('tweets/' + userId + "/").push({ tweet: twit.value, info: twitInfo.value, img: twitImg.innerHTML, color: twitColor.value, size: twitSize.value, time: js_time });

			twit.value = "";
			twitInfo.value = "";
			twitImg.innerHTML = "";
			twitColor.value = "";
			twitSize.value = "";
			error.innerHTML = "";

			console.log("tweet written")

			firebase.database().ref('/tweets/' + userId).once('value').then(function (snapshot) {
				var data = (snapshot.val());
				var keys = [];

				if (data == null) {
					console.log("No data found at /tweets");
				} else {
					snapshot.forEach(function (childSnapshot) {
						keys.push(childSnapshot.key);
					});

					updatetweets(keys, data);
				}
			});
			// The unique key stored in tweetid is based on a timestamp, so list items will automatically be ordered chronologically. Because Firebase generates a unique key for each tweet, no write conflicts will occur if multiple users add a post at the same time. https://firebase.google.com/docs/database/admin/save-data
		}

		function like(key, i) {
			var like = document.getElementById("like" + i.toString());

			like.innerHTML = String(Number(like.innerHTML) + 1);

			console.log(userId);
			console.log(key + " / " + i.toString() + " / " + like.innerHTML);

			firebase.database().ref("tweets/" + userId + "/" + key + "/likesCount").set(like.innerHTML);
		}
        
        function dislike(key, i) {
			var dislike = document.getElementById("dislike" + i.toString());

			dislike.innerHTML = String(Number(dislike.innerHTML) + 1);

			console.log(userId);
			console.log(key + " / " + i.toString() + " / " + dislike.innerHTML);

			firebase.database().ref("tweets/" + userId + "/" + key + "/dislikesCount").set(dislike.innerHTML);
		}

		function signin() {
			console.log("Signing in");
			var provider = new firebase.auth.GoogleAuthProvider();
			firebase.auth().signInWithRedirect(provider).then(function (result) {
				document.getElementById("divLoading").classList.add("hidden");
				document.getElementById("divLogin").classList.add("hidden");
				document.getElementById("divTweet").classList.remove("hidden");
			});
		}

		function signout() {
			console.log("Signing out");
			firebase.auth().signOut().then(function () {
				document.getElementById("divLoading").classList.add("hidden");
				document.getElementById("divLogin").classList.remove("hidden");
				document.getElementById("divTweet").classList.add("hidden");
			});
		}

		function encodeImageFileAsURL() {
			var filesSelected = document.getElementById("twitFile").files;

			if (filesSelected.length > 0) {
				var fileToLoad = filesSelected[0];
				var fileReader = new FileReader();

				fileReader.onload = function (fileLoadedEvent) {
					var srcData = fileLoadedEvent.target.result;

					document.getElementById("twitImg").innerHTML = srcData;
					console.log(srcData);
				}

				fileReader.readAsDataURL(fileToLoad);
			}
		} // end function

	</script>
</Body>
</html>

______________________________________________________________________________________________________________________________
<html>
<link rel="stylesheet" href="Main.css">
    <div class="content">
    <div class="pill-nav">
		<a href="home.html">Home</a>
        <a href="catalog.html">Catalog</a>
		<a class="active" href="about.html">About</a>
	</div>
<style>
.Jack {
  height: 300px;
  background-color: #8c8c8c;
  border-style: dotted;
  border-color: black;
  color: white;
  padding: 16px 32px;
  text-align: center;
  font-size: 16px;
  margin: 4px 2px;
  opacity: 0.6;
  transition: 0.3s;
}
    .Robert {
  background-color: #8c8c8c;
  border-style: dotted;
  border-color: black;
  color: white;
  padding: 16px 32px;
  text-align: center;
  font-size: 16px;
  margin: 4px 2px;
  opacity: 0.6;
  transition: 0.3s;
}

    
.Jack:hover {opacity: 1}
.Robert:hover {opacity: 1}


    
</style>
<body>
<div class="About"> <h1 style="font-size: 100px; strong; text-align: center;">Meet the creators</h1></div>
<div class="About"><h2 style="font-size: 60px; text-align: center">The Mission</h2></div>
<center><div style="background-color: #8c8c8c; border-style: dotted; border-color: black; color: white; width: 900px; align-content: center"><br>Founded by two shoe-loving high schoolers, Flyfeet strives to provide the perfect piece of footware, any type, any place, any time. <br><br> -Robert Muroni and Jack Wildi</div></center>
<div class="About"><h2 style="font-size: 60px; text-align: center">Meet the team</h2></div>
  <div style="display: flex">
    <div style="align-content: left; width: 50%; height: 300px;" class="Jack">
    <img style="float: left;height: 200px; width: 300px;" src="Jack.jpg">
       <div><h1>About Jack</h1>Email - Jack.Wildi@ucc.on.ca <br><br>
Linkedin - https://www.linkedin.com/in/jack-wildi-13bb9814b/?originalSubdomain=ca<br><br>

Jack has been a student at Upper Canada College for 2 years and has been enrolled in computer science for the duration of his time at the college. Jack loves sports, and more importantly the shoes that he wears when playing the sport. Shoes have always been a big part of Jacks life no matter what. </div>
   </div>
    
    <div style=" width: 380px; padding-top: 40px;" class="Jack">
        <img style="width: 300px; height: 200px;" src="Ucc.jpg">
    </div>
</div>
    
    <br>
<div style="display: flex">
     <div style="align-content: left; width: 50%; height: 300px;" class="Robert">
        <img style="float: left; height: 200px; width: 300px " src="Robert.jpg">
       <div><h1>About Robert</h1>Email: Rober.Muroni@ucc.on.ca <br><br> Linkedin: https://www.linkedin.com/in/robert-muroni-60b125150/ <br><br> <br><br>A passionate sports fan, Robert has been a student at Upper Canada College since grade 3. Over the years, Robert has developped a passion for the Toronto Maple Leafs and the Toronto Raptors. "I've come to understand the basketball fans are crazy about their shoes" stated Robert "and my goal is to help people who are crazy about shoes find the perfect pair" </div>
    </div>
<div style=" width: 380px; padding-top: 40px;" class="Jack">
        <img style="width: 300px; height: 200px;" src="Feet.jpg">
    </div>
</div>    
</body>
</html>
______________________________________________________________________________________________________________________________
<html>
<head>
  <link rel="stylesheet" href="Main.css">
  <link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
  <link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
</head>
<Body>
    <div class="content">
    <div class="pill-nav">
		<a class="active" href="home.html">Home</a>
		<a href="catalog.html">Catalog</a>
		<a href="about.html">About</a>
	</div>

	<script src="https://www.gstatic.com/firebasejs/5.7.3/firebase.js"></script>
	<script src="https://apis.google.com/js/platform.js" async defer></script>
    
	<div class="w3-container w3-center w3-animate-top">
	<h1 style="strong; text-align: center;">Welcome to Flyingfeet</h1>
	</div>
    <div class="w3-container w3-center w3-animate-left">
    <h2 style="text-align: center">The number one place to browse shoes</h2>
        
	<div class="slideshow-container">

	  <!-- Full-width images with number and caption text -->
	  <div class="mySlides fade">
		<div class="numbertext">1 / 3</div>
		<img src="shoe1.5.jpg" style="width:500px; height: 400px; padding: 10px">
		<div class="text">Adidas Yeezy Boost 350 V2 Static </div>
	  </div>

	  <div class="mySlides fade">
		<div class="numbertext">3 / 3</div>
		<img src="shoe2.5%20copy.jpg" style="width:500px; height: 400px; padding: 10px ">
		<div class="text">Nike Kyrie S1Hybrid 12</div>
	  </div>

	  <div class="mySlides fade">
		<div class="numbertext">2 / 3</div>
		<img src="shoe3.5%20copy.jpg" style="width:500px; height: 400px; padding: 10px">
		<div class="text">Nike LeBron Soldier 12</div>
	  </div>
    
	  </div>

	  <!-- Next and previous buttons -->
	  <a class="prev" onclick="plusSlides(-1)">&#10094;</a>
	  <a class="next" onclick="plusSlides(1)">&#10095;</a>
	</div>
	<br>

	<!-- The dots/circles -->
	<div style="text-align:center">
	  <span class="dot" onclick="currentSlide(1)"></span> 
	  <span class="dot" onclick="currentSlide(2)"></span> 
	  <span class="dot" onclick="currentSlide(3)"></span> 
	</div>
        
	<div>
 </div>

	<script>
		var slideIndex = 0;

		autoSlide();

		// Next/previous controls
		function plusSlides(n) {
		  showSlides(slideIndex += n);
		}

		// Thumbnail image controls
		function currentSlide(n) {
		  showSlides(slideIndex = n);
		}

		function showSlides(n) {
			var i;
			var slides = document.getElementsByClassName("mySlides");
			var dots = document.getElementsByClassName("dot");
			if (n > slides.length) {slideIndex = 1} 
			if (n < 1) {slideIndex = slides.length}
			for (i = 0; i < slides.length; i++) {
				slides[i].style.display = "none"; 
			}
			for (i = 0; i < dots.length; i++) {
				dots[i].className = dots[i].className.replace(" active", "");
			}
			slides[slideIndex-1].style.display = "block"; 
			dots[slideIndex-1].className += " active";
		}

		function autoSlide() {
			plusSlides(1);
			setTimeout(autoSlide, 4000); // Change image every 2 seconds
		}
		</script>
    </body>
</html>

