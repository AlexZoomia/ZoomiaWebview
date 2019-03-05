<template>
	<view class="container"  v-if="con_reload">
		<text class="error name">
Connection Problem
		</text>
		<text class="error">
The connection to server is failed, 
or the network connection is lost.
		</text>
		<touchable-opacity class="btn" :on-press="reload">
			<text class="text">Reload</text>
	    </touchable-opacity>
	</view>
	<view class="container"  v-else-if="net_reload">
		<text class="error name">
Network Error
		</text>
		<text class="error">
The network connection is lost.
		</text>
		<touchable-opacity class="btn" :on-press="reload">
			<text class="text">Reload</text>
	    </touchable-opacity>
	</view>
	<web-view class="web" v-else
		:key="this.web"
        :source= "{uri: 'https://my.zoomiya.com'}"
        :ref="(ref) => { this.webview = ref }"
        :onShouldStartLoadWithRequest="(event) => onShould(event)"
        :bounces= "false"
        :startInLoadingState="true"
        :onError="() => conError()"  
        :renderError="() => conError()"
        :onNavigationStateChange="(event) => onChange(event)"

    />
</template>


<script>
import React, {Component} from 'react';
import {WebView, Linking, NetInfo, StatusBar, ActivityIndicator} from 'react-native';
import URI from 'urijs';

export default {
	data(){
		return {
			// web: 'this.web-view',
			key: 0,
			// ref: this.ref,
			con_reload: false,
			net_reload: false,
		}
	},
	methods: {
		conError: function () {
			this.con_reload = true;
		},
		netError: function () {
			this.net_reload = true;
		},
		reload: function() {
			this.key = !this.key;
			console.log(this.key);
			// webView.reload();
			this.con_reload = false;
			this.net_reload = false;
		},
		alerting: function() {
			alert(":(");
		},
		onShould: function(event){
			var uri1 = new URI("https://my.zoomiya.com");
      		var uri2 = new URI(event.url);
      		console.log(StatusBar.currentHeight);
      		if (uri1.hostname() !== uri2.hostname()) {
          		Linking.openURL(event.url)
          		return false
      		}
      		return true
		},
		onChange: function(event){
			var uri1 = new URI("https://my.zoomiya.com");
			var uri2 = new URI(event.url);
			NetInfo.getConnectionInfo().then((connectionInfo) => {
				if (connectionInfo.type == 'wifi') {
					this.netError();
				}
			});
			if (uri1.hostname() !== uri2.hostname()) {
				Linking.openURL(event.url);
				this.webview.stopLoading();
				// this.webview.reload();
			}
		},
		// alerting() {
		// 	var x = window.location.host;
//          	alert(this.x);
//      	}
	},
	 beforeMount(){
		NetInfo.getConnectionInfo().then((connectionInfo) => {
			if (connectionInfo.type == 'wifi') {
				this.netError();
			}
		});
		function handleFirstConnectivityChange(connectionInfo) {
  			console.log(
    			'First change, type: ' +
      			connectionInfo.type +
      			', effectiveType: ' +
      			connectionInfo.effectiveType,
 			 );
  			NetInfo.removeEventListener(
   				'connectionChange',
    			handleFirstConnectivityChange,
  			);
		}
		NetInfo.addEventListener('connectionChange', handleFirstConnectivityChange);
	}
}
</script>
 
<style>
.container {
	flex: 1;
	align-items: center;
	justify-content: center;
	background-color: #f5f5f5;
}
.error {
	/*align-self: auto;*/
	text-align: center;
}
.name {
	font-weight: bold;
}
.btn {
	background-color: #065fd4;
	padding: 20;
	padding-left: 50;
	padding-right: 50;
	border-radius: 5;
	margin-top: 20;
}
.text {
	color: white;
}
.web {
	margin-top: 20;
}
</style> 
