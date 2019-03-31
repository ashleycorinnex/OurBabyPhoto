<template>
    <Page>
        <ActionBar>
            <StackLayout orientation="horizontal">
            <Image src="~/assets/images/logo.png" width="40" height="40" verticalAlignment="center" class="p-r-15" />
            <Label text="BabyDNA" fontSize="24" verticalAlignment="center" />
            </StackLayout>
        </ActionBar>
        <TabView id="tabViewContainer" :selectedIndex="selectedIndex">
            <TabViewItem title="Start">
                <StackLayout verticalAlignment="center">
                    <Label height="20%" /> 
                    <Label class="p-l-15 p-r-15 p-t-15 p-b-15" height="20%" fontSize="16" horizontalAlignment="center" textWrap="true" text="Find out the DNA your baby will have!" />
                    <Button class="p-t-15" height="20%" backgroundColor="#53ba82" color="#fff" text="Begin"  width="50%"  @tap="selectedIndex = 1" />
                     <Label height="20%" /> 
                </StackLayout>
            </TabViewItem>
            <TabViewItem title="Mother">
                <StackLayout>
                    <Image :src="mother.src || '~/assets/images/woman.jpg'" height="80%" stretch="aspectFit"  class="img-rounded p-l-15 p-r-15 p-t-15" @tap="choosePhoto(true)"></Image>
                    <FlexboxLayout v-if="!mother.src" height="20%">
                        <Button text="Choose Photo" width="50%" @tap="choosePhoto(true)" />
                        <Button text="Take Photo" color="#000" width="50%" @tap="takePhoto(true)"  />
                    </FlexboxLayout>
                    <FlexboxLayout v-else height="20%">
                        <Button text="Reset" width="50%" @tap="mother = {}" />
                        <Button text="Next" backgroundColor="#53ba82" color="#fff" width="50%" @tap="selectedIndex = 2"  />
                    </FlexboxLayout>
                </StackLayout>
            </TabViewItem>
            <TabViewItem title="Father">
                <StackLayout>
                    <Image :src="father.src || '~/assets/images/man.jpg'" height="80%" stretch="aspectFit"  class="img-rounded p-l-15 p-r-15 p-t-15" @tap="choosePhoto(false)"></Image>
                    <FlexboxLayout v-if="!father.src" height="20%">
                        <Button text="Choose Photo" width="50%" @tap="choosePhoto(false)" />
                        <Button text="Take Photo" color="#000" width="50%" @tap="takePhoto(false)"  />
                    </FlexboxLayout>
                    <FlexboxLayout v-else height="20%">
                        <Button text="Reset" width="50%" @tap="father = {}" />
                        <Button text="Finish" backgroundColor="#53ba82" color="#fff" width="50%" @tap="showResult"  />
                    </FlexboxLayout>
                </StackLayout>
            </TabViewItem>
        </TabView>
    </Page>
</template>

<script>
 import * as camera from "nativescript-camera";
 import * as imagepicker from "nativescript-imagepicker";
 import { Image } from "tns-core-modules/ui/image";

  export default {
    data() {
      return {
          selectedIndex: 0,
          mother: {},
          father: {},
          detail: {
            data(){
                return{
                    baby: {},
                }
            },
            created(){
                fetch("https://45je41wsq9.execute-api.us-west-2.amazonaws.com/default/GetBabyPhotoSettings", {
                    headers:  new Headers({
                        'x-api-key': 'Bqfpv4qSz28Af5i8vYdrT1AXw9RYccZp1dJYwlps'
                    })
                })
                .then((response) => response.text())
                .then((r) => {
                    var obj = JSON.parse(r);
                    let img = new Image();
					img.src = obj.Item.Urls.SS[(Math.floor(Math.random() * obj.Item.Urls.SS.length + 1))];
                    this.baby = img;
                }).catch((e) => {
                });
            },
            template: `
                <Page>
                <ActionBar>
                    <StackLayout orientation="horizontal">
                    <Image src="~/assets/images/logo.png" width="40" height="40" verticalAlignment="center" class="p-r-15" />
                    <Label text="BabyDNA" fontSize="24" verticalAlignment="center" />
                    </StackLayout>
                </ActionBar>
                <StackLayout> 
                    <Label class="p-l-15 p-r-15 p-t-15 p-b-15" height="20%" fontSize="16" horizontalAlignment="center" textWrap="true" text="Your Baby's DNA Results" />
                    <Image :src="baby.src" stretch="aspectFit" height="60%"  class="img-rounded p-l-15 p-r-15 p-t-15"></Image>
                    <Button text="Start Over" height="20%" backgroundColor="#53ba82" color="#fff" @tap="$navigateBack" />
                </StackLayout>
                </Page>
            `
            }
        }
    },
    methods:{
        choosePhoto(isMother){
            let context = imagepicker.create({
                mode: "single"
            });

            context.authorize()
			.then(function() {
				return context.present();
			})
			.then(selection => {
				selection.forEach(selected => {
					let img = new Image();
                    img.src = selected;
                    if(isMother)
                        this.mother = img;
                    else
                        this.father = img;
                });
			}).catch(function (e) {
				console.log('error in selectPicture', e);
			});
        },
        takePhoto(isMother) {
			camera.requestPermissions()
			.then(() => {
				camera.takePicture({ width: 300, height: 300, keepAspectRatio: true, saveToGallery:false })
				.then(imageAsset => {
					let img = new Image();
					img.src = imageAsset;
                    if(isMother)
                        this.mother = img;
                    else
                        this.father = img;
				})
				.catch(e => {
					console.log('error:', e);
				});
			})
			.catch(e => {
				console.log('Error requesting permission');
			});
        },
        showResult(){
            if(this.mother.src && this.father.src){
                this.$navigateTo(this.detail);
                this.selectedIndex = 0;
                this.mother = {};
                this.father = {};
            }
        },
    }
  }
</script>

<style>
    ActionBar {
        background-color: #53ba82;
        color: #ffffff;
    }
</style>
