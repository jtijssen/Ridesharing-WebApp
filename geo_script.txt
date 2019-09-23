var myLat = 0;
var myLng = 0;
var me = new google.maps.LatLng(myLat, myLng);
var myOptions = {
  zoom: 16,
  center: me,
  mapTypeId: google.maps.MapTypeId.ROADMAP
};
var map;
var marker;
var infowindow = new google.maps.InfoWindow();
var pass_or_veh;
var weinermobile_string;
var weiner_dist;
var other_dist = new Array();

function getMyLocation() {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(function(position) {
      myLat = position.coords.latitude;
      myLng = position.coords.longitude;
      renderMap();
    });
  }
  else {
    alert("Geolocation is not supported by your web browser.");
  }
}

function renderMap() {
  me = new google.maps.LatLng(myLat, myLng);
    
  map.panTo(me);
  var my_content;
  var image = 'pin_personal.png'
        
  marker = new google.maps.Marker({
    position: me,
    icon: image
  });
  marker.setMap(map);
}

function getMiles(i) {
     return i*0.000621371192;
}

function http_req() {
  if (navigator.geolocation) { 
      navigator.geolocation.getCurrentPosition(function(position) {
      myLat = position.coords.latitude;
      myLng = position.coords.longitude;
      var http = new XMLHttpRequest();
      var url = 'https://limitless-scrubland-48566.herokuapp.com/rides';
      var params = 'username=PG764R7x&lat=' + myLat + '&lng=' + myLng;
      http.open('POST', url, true);

      http.setRequestHeader("Content-type", "application/x-www-form-urlencoded");

      http.onreadystatechange = function() {
        if(http.readyState == 4 && http.status == 200) {
          var obj_data = JSON.parse(http.responseText);
          var i;
          var sum = 0;
          var weiner = false;
          let mark = new Array();
          var temp_lat;
          var least_val;
          var user = new Array();
          var count = 0;
          var weiner_img = "weinermobile.png"
          var car_img = "car.png"
          var pass_img = "passenger.png"
          if ("passengers" in obj_data){
            pass_or_veh = "passenger"
            for(i = 0; i < obj_data.passengers.length; i++){
              temp_lat = obj_data.passengers[i].lat;
              temp_lng = obj_data.passengers[i].lng;
              if (obj_data.passengers[i].username == 'WEINERMOBILE'){
                weiner = true;
                weiner_dist = google.maps.geometry.spherical.computeDistanceBetween(me, new google.maps.LatLng(temp_lat, temp_lng)); 
                mark[i] = new google.maps.Marker({
                  position: {lat: temp_lat, lng: temp_lng},
                  icon: weiner_img,
                  map: map,
                  title: "" + getMiles(weiner_dist)
                });
                google.maps.event.addListener(mark[i], 'click', function() {
                   other_content = "<p>Username: WEINERMOBILE</p>" +
                    "<p>Distance from you: " + this.getTitle() + " miles </p>";
                  infowindow.setContent(other_content);
                  infowindow.open(map, this);
                });
              }
              else{
                mark[i] = new google.maps.Marker({
                  position: {lat: temp_lat, lng: temp_lng},
                  icon: pass_img,
                  map: map,
                  title: "" + (count+1)
                });
                user[count] = obj_data.passengers[i].username;
                other_dist[count] = google.maps.geometry.spherical.computeDistanceBetween(me, new google.maps.LatLng(temp_lat, temp_lng)); 
                count += 1;
                google.maps.event.addListener(mark[i], 'click', function() {
                   other_content = "<p>Username: " + user[Number(this.getTitle()) - 1] + "</p>" +
                    "<p>Distance from you: " + getMiles(other_dist[Number(this.getTitle()) - 1]) + " miles </p>";
                  infowindow.setContent(other_content);
                  infowindow.open(map, this);
                });
              }
            }
          }
          if ("vehicles" in obj_data){
            pass_or_veh = "vehicle"
            for(i = 0; i < obj_data.vehicles.length; i++){
              temp_lat = obj_data.vehicles[i].lat;
              temp_lng = obj_data.vehicles[i].lng;
              if (obj_data.vehicles[i].username == 'WEINERMOBILE'){
                weiner = true;
                weiner_dist = google.maps.geometry.spherical.computeDistanceBetween(me, new google.maps.LatLng(temp_lat, temp_lng)); 
                mark[i] = new google.maps.Marker({
                  position: {lat: temp_lat, lng: temp_lng},
                  icon: weiner_img,
                  map: map,
                  title: "" + getMiles(weiner_dist)
                });
                google.maps.event.addListener(mark[i], 'click', function() {
                   other_content = "<p>Username: WEINERMOBILE</p>" +
                    "<p>Distance from you: " + this.getTitle() + " miles </p>";
                  infowindow.setContent(other_content);
                  infowindow.open(map, this);
                });
              }
              else{
                mark[i] = new google.maps.Marker({
                  position: {lat: temp_lat, lng: temp_lng},
                  icon: car_img,
                  map: map,
                  title: "" + (count+1)
                });
                user[count] = obj_data.vehicles[i].username;
                other_dist[count] = google.maps.geometry.spherical.computeDistanceBetween(me, new google.maps.LatLng(temp_lat, temp_lng)); 
                count += 1;
                google.maps.event.addListener(mark[i], 'click', function() {
                  other_content = "<p>Username: " + user[Number(this.getTitle()) - 1] + "</p>" +
                  "<p>Distance from you: " + getMiles(other_dist[Number(this.getTitle()) - 1]) + " miles </p>";
                  infowindow.setContent(other_content);
                  infowindow.open(map, this);
                });
              }
            }
          }
          if(count == 0) {
            least_val = "No " + pass_or_veh + "s found"
          }
          else{
            least_val = other_dist[0];
            for(i = 1; i < count; i++){
              if(least_val > other_dist[i]){
                least_val = other_dist[i];
              }
            }
          }
          var miles_least_val = getMiles(least_val);
          var weiner_dist_miles = getMiles(weiner_dist);
          google.maps.event.addListener(marker, 'click', function() {
          infowindow.setContent(my_content);
          infowindow.open(map, marker);
            });
          if(weiner){
              weinermobile_string = "The Weinermobile is " + weiner_dist_miles + " miles away from me!"
            }
          else {
              weinermobile_string = "The Weinermobile is nowhere to be seen..."
            }
          my_content = "<p>Username: PG764R7x</p>" +
                      "<p>Distance to nearest " + pass_or_veh + ": " + miles_least_val +" miles</p>" +
                      weinermobile_string;
        }
      }
      http.send(params);
    });
  }
}

function init() {
  map = new google.maps.Map(document.getElementById("map_canvas"), myOptions);
  getMyLocation();
  http_req();
}

