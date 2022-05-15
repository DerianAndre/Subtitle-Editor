<template>
	<div class="container">
		<div class="card shadow">
			<div class="card-body">
				<form method="post" enctype="multipart/form-data" v-if="subtitleArray.length == 0">
					<div class="d-flex flex-column align-items-center" >
						<div class="mb-3">
							<label for="formFileLg" class="form-label">Elegir</label>
							<input id="inputFiles" class="form-control form-control-lg" type="file" name="files[]" 
								accept=".ass"
								@change="isLoading = true, handleForm($event)" multiple>
						</div>
						<button class="btn btn-primary btn-submit btn-lg" type="button">
							Editar subtitulos
						</button>

						<div class="d-flex justify-content-center" v-if="isLoading">
							<div class="spinner-border" role="status">
								<span class="visually-hidden">Loading...</span>
							</div>
						</div>
					</div>
				</form>
				<div v-else>
					<div v-for="item, index in subtitleArray" :key="index">
						<h3>{{ item["File Name"]}}</h3>
						<pre>{{ JSON.stringify(item, null, "\t") }}</pre>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				fileList: [],
				subtitleArray: [],
				subArray: [],
				isLoading: false,
			}
		},
		methods: {
			async handleForm(event) {
				this.fileList = event.target.files;
				for(let i = 0; i < this.fileList.length; i++) {
					var content = await this.readFile(this.fileList[i]);
					this.subtitleArray[i] = {
						"File Name": this.fileList[i].name,
						"File Size": this.fileList[i].size,
						"Raw": content.replaceAll("\r", "")
					}
				}
				this.isLoading = false;
				this.subToJson();
			},
			async readFile(file) {
				var fileReader = new FileReader();
				return new Promise((resolve, reject) => {
					fileReader.onerror = () => {
						fileReader.abort();
						reject(new DOMException("Problem parsing input file."));
					};
					fileReader.onload = (event) => {
						resolve(event.target.result);
					}
					fileReader.readAsText(file);
				});
			},
			subToJson() {
				var subtitleArray = this.subtitleArray;
				const regex = /^(\[.*?\])$\n^(.*?)\n$/gsm;	
				for(let i = 0; i < subtitleArray.length; i++) {
					let matches;
					while ((matches = regex.exec(subtitleArray[i].Raw)) !== null) {
						switch (matches[1]) {
							case "[Script Info]":
								subtitleArray[i]["Script Info"] = matches[2];
								break;
							case "[Aegisub Project Garbage]":
								subtitleArray[i]["Aegisub Project Garbage"] = matches[2];
								break;
							case "[V4+ Styles]":
								subtitleArray[i]["V4+ Styles"] = matches[2];
								break;
							case "[V4 Styles]":
								subtitleArray[i]["V4 Styles"] = matches[2];
								break;
							case "[Events]":
								subtitleArray[i]["Events"] = matches[2];
								break;
							default:
								subtitleArray[i]["Unknown"] = matches[2];
								break;
						}
					}
				}
				this.subtitleArray = subtitleArray;
			}
		},
	}
</script>

<style>

</style>