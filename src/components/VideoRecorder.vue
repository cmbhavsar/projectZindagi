<template>
		<div>
			<header class="header">
				<h1>Video Recorder</h1>
				<p>Record a video and upload it to the server.</p>
			</header>
			<div class="video-container">
				<div v-if="cameraAccess || startingCamera" class="video-frame">
					<video ref="videoPlayer" class="video-player" autoplay></video>
				</div>
				<div v-if="recordedVideoBlob" class="preview-frame">
					<p>Recording Preview:</p>
					<video class="preview-player" :src="recordedVideoBlob" controls></video>
				</div>
				<div class="controls">
					<button @click="toggleCamera" class="control-button">{{ cameraAccess ? 'Stop Camera' : 'Start Camera' }}</button>
					<button @click="startRecording" :disabled="recording || !cameraAccess" class="control-button">Start Recording</button>
					<button @click="stopRecording" :disabled="!recording" class="control-button">Stop Recording</button>
					<button @click="uploadVideo" :disabled="!recordedVideoBlob || uploading" class="control-button">Upload Video</button>
				</div>
			</div>
		</div>
	</template>
	
	<script>
	import { ref } from 'vue';
	
	export default {
		setup() {
			const videoPlayer = ref(null);
			const mediaStream = ref(null);
			const mediaRecorder = ref(null);
			const recording = ref(false);
			const recordedVideoBlob = ref(null);
			const cameraAccess = ref(false);
			const startingCamera = ref(false);
			const uploading = ref(false);
	
			const startCamera = async () => {
				try {
					startingCamera.value = true;
					mediaStream.value = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
					videoPlayer.value.srcObject = mediaStream.value;
					cameraAccess.value = true;
				} catch (error) {
					console.error('Error starting camera:', error);
				} finally {
					startingCamera.value = false;
				}
			};
	
			const stopCamera = () => {
				if (mediaStream.value) {
					mediaStream.value.getTracks().forEach(track => track.stop());
					videoPlayer.value.srcObject = null;
					cameraAccess.value = false;
				}
			};
	
			const toggleCamera = () => {
				if (cameraAccess.value) {
					stopCamera();
				} else {
					startCamera();
				}
			};
	
			const startRecording = () => {
				recordedVideoBlob.value = null;
				mediaRecorder.value = new MediaRecorder(mediaStream.value);
				const chunks = [];
	
				mediaRecorder.value.ondataavailable = (event) => {
					if (event.data.size > 0) {
						chunks.push(event.data);
					}
				};
	
				mediaRecorder.value.onstop = () => {
					recordedVideoBlob.value = URL.createObjectURL(new Blob(chunks, { type: 'video/webm' }));
				};
	
				mediaRecorder.value.start();
				recording.value = true;
			};
	
			const stopRecording = () => {
				if (mediaRecorder.value && mediaRecorder.value.state !== 'inactive') {
					mediaRecorder.value.stop();
					recording.value = false;
					stopCamera(); // Stop camera when recording stops
				}
			};
	
			const uploadVideo = async () => {
				if (!recordedVideoBlob.value || uploading.value) return;
	
				uploading.value = true;
	
				const formData = new FormData();
				formData.append('video', recordedVideoBlob.value);
	
				try {
					const response = await fetch('YOUR_API_ENDPOINT', {
						method: 'POST',
						body: formData,
						headers: {
							'Content-Type': 'multipart/form-data',
						}
					});
	
					if (response.ok) {
						const data = await response.json();
						console.log(data);
					} else {
						console.error('Failed to upload video:', response.status);
					}
				} catch (error) {
					console.error('Error uploading video:', error);
				} finally {
					uploading.value = false;
				}
			};
	
			return {
				videoPlayer,
				recording,
				recordedVideoBlob,
				cameraAccess,
				startingCamera,
				toggleCamera,
				startRecording,
				stopRecording,
				uploadVideo,
			};
		},
	};
	</script>
	
	<style scoped>
	.header {
		text-align: center;
		margin-bottom: 20px;
	}
	
	.video-container {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.video-frame {
		border: 1px solid #ccc;
		border-radius: 8px;
		overflow: hidden;
		margin-bottom: 20px;
	}
	
	.preview-frame {
		border: 1px solid #ccc;
		border-radius: 8px;
		overflow: hidden;
		margin-top: 20px;
	}
	
	.video-player {
		width: 100%;
	}
	
	.preview-player {
		width: 100%;
	}
	
	.controls {
		display: flex;
		justify-content: center;
		margin-top: 10px;
	}
	
	.control-button {
		margin: 0 10px;
		padding: 10px 20px;
		font-size: 16px;
		cursor: pointer;
	}
	
	@media (max-width: 768px) {
		.video-container {
			padding: 0 20px;
		}
	}
	</style>
	