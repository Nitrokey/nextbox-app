<template>
	<div>
		<span :class="'tag ' + showState">
			<span :class="'tag-icon ' + showIcon" />
			<span class="tag-text" v-html="showText"></span>
			<span class="tag-middle" />
			<span class="tag-extra">{{ showExtra }}</span>
		</span>
		<span class="tag-help" 
			v-if="showState === 'error'" 
			v-html="showHelp" />
	</div>
</template>


<script>

//import '@nextcloud/dialogs/styles/toast.scss'
import { generateUrl } from '@nextcloud/router'
//import { showError, showSuccess } from '@nextcloud/dialogs'
import { showError } from '@nextcloud/dialogs'
import axios from '@nextcloud/axios'

export default {
	name: 'StatusBar',

	components: { },

	props: {
		icon: { type: String, default: null },
		state: { type: String, default: 'neutral' },
		text: { type: String, default: '' },
		help: { type: String, default: '' },
		extra: { type: String, default: '' },
		// all of the above within one object:
		status: { type: Object, default: () => {} },
		// use one of the presets 
		preset: { type: String, default: '' },
	},

	data() {
		return {
			internal: {
				icon: null,
				state: 'neutral',
				text: '',
				help: '',
				extra: '',
			}
		}
	},

	computed: { 
		showText() {
			const obj = this.status || this.internal
			return (obj && 'text' in obj) ? obj.text : this.text
		},
		showIcon() {
			const obj = this.status || this.internal
			// shall be one of 'success', 'error', 'warning' and 'neutral'
			let dynamic = 'loading-small'
			// set icon based on 'this.internal' member 'icon'
			if (obj && 'icon' in obj && obj.icon) {
				dynamic = obj.icon
			// set icon based on property 'icon'
			} else if (this.icon !== null) {
				dynamic = this.icon
			// none of the above apply, so map 'this.showState' to default icon set
			} else {
				if (this.showState === 'success') dynamic = 'checkmark'
				else if (this.showState === 'error') dynamic = 'error'
				else if (this.showState === 'warning') dynamic = 'stop'
			} 
			return 'icon icon-' + dynamic
		},
		showState() {
			const obj = this.status || this.internal
			return (obj && 'state' in obj) ? obj.state : this.state
		},
		showExtra() {
			const obj = this.status || this.internal
			return (obj && 'extra' in obj) ? obj.extra : this.extra
		},
		showHelp() {
			const obj = this.status || this.internal
			return (obj && 'help' in obj) ? obj.help : this.help
		},
	},

	async mounted() { 
		if (this.preset === 'resolve_ipv4') {
			this.resolve_ipv4()
		} else if (this.preset === 'resolve_ipv6') {
			this.resolve_ipv6()
		} else if (this.preset === 'reach_http') {
			this.reach_http()
		} else if (this.preset === 'reach_https') {
			this.reach_https()
		} else if (this.preset === 'reach_proxy') {
			this.reach_proxy()
		} else {
			this.internal = {
				text: this.text,
				icon: this.icon,
				help: this.help,
				extra: this.extra,
				state: this.state,
			}
		}
	},
	
	methods: { 
		async resolve_ipv4() {
			this.internal.text = 'DNS (IPv4) resolve testing pending'
			this.internal.help = 'Please see the following topics at <a class="bold" target="_blank" href="https://docs.nitrokey.com/nextbox/">docs.nitrokey.com/nextbox</a> to troubleshoot: '
					  + '<span class="bold">(Dynamic) DNS</span>'
			// get ipv4 resolve
			axios.get(generateUrl('/apps/nextbox/forward/dyndns/test/resolve/ipv4')).then((res) => {
				if (res.data.result === 'success') {
					this.internal.state = 'success'
					this.internal.icon = 'checkmark'
					this.internal.text = `Successfully resolved: ${res.data.data.domain} to: ${res.data.data.ip}`
					
				} else {
					let suffix = ''
					if (res.data.data) {
						suffix = `need: ${res.data.data.ip} found: ${res.data.data.resolve_ip}`
					}
					this.internal.state = 'error'
					this.internal.icon = 'close'
					this.internal.text = `Failed resolving: ${res.data.data.domain} [IPv4] ${suffix}`
				}
			}).catch((e) => {
				console.error(e)
				showError(t('nextbox', 'Connection Failed'))
			})
		},
		async resolve_ipv6() {
			this.internal.text = 'DNS (IPv6) resolve testing pending'
			this.internal.help = 'Please see the following topics at <a class="bold" target="_blank" href="https://docs.nitrokey.com/nextbox/">docs.nitrokey.com/nextbox</a> to troubleshoot: '
					  + '<span class="bold">DNS-Rebind Protection</span>, <span class="bold">(Dynamic) DNS</span>'
			// get ipv6 resolve
			axios.get(generateUrl('/apps/nextbox/forward/dyndns/test/resolve/ipv6')).then((res) => {
				if (res.data.result === 'success') {
					this.internal.state = 'success'
					this.internal.icon = 'checkmark'
					this.internal.text = `Successfully resolved: ${res.data.data.domain} to: ${res.data.data.ip}`
					
				} else {
					let suffix = ''
					if (res.data.data) {
						suffix = `need: ${res.data.data.ip} found: ${res.data.data.resolve_ip}`
					}
					this.internal.state = 'error'
					this.internal.icon = 'close'
					this.internal.text = `Failed resolving: ${res.data.data.domain} [IPv6] ${suffix}`
				}
			}).catch((e) => {
				console.error(e)
				showError(t('nextbox', 'Connection Failed'))
			})
		},
		async reach_http() {
			this.internal.text = 'Reachability waiting to be tested'
			this.internal.help = 'Please see the following topics at <a class="bold" target="_blank" href="https://docs.nitrokey.com/nextbox/">docs.nitrokey.com/nextbox</a> to troubleshoot: '
					  + '<span class="bold">Port-Forwarding/Firewall router settings</span>, <span class="bold">(Dynamic) DNS</span>'
			// get general (http) reachability
			axios.get(generateUrl('/apps/nextbox/forward/dyndns/test/http')).then((res) => {
				if (res.data.result === 'success') {
					this.internal.state = 'success'
					this.internal.icon = 'checkmark'
					this.internal.text = `Successfully tested reachability for: ${res.data.data.domain}`
				
				} else {
					this.internal.state = 'error'
					this.internal.icon = 'close'
					this.internal.text = `Failed reachability for: ${res.data.data.domain}`
				}
			}).catch((e) => {
				console.error(e)
				showError(t('nextbox', 'Connection Failed'))
			})
		},
		async reach_https() {
			// get general (http6) reachability
			axios.get(generateUrl('/apps/nextbox/forward/dyndns/test/https')).then((res) => {
				if (res.data.result === 'success') {
					this.internal.state = 'success'
					this.internal.icon = 'checkmark'
					this.internal.text = `Successfully tested reachability for: ${res.data.data.domain}`
					
				} else {
					this.internal.state = 'error'
					this.internal.icon = 'close'
					this.internal.text = `Failed reachability for: ${res.data.data.domain}`
					
				}
			}).catch((e) => {
				console.error(e)
				showError(t('nextbox', 'Connection Failed'))
			})
		},
		async reach_proxy() {
			this.internal.text = 'Quickstart remote access test pending'
			this.internal.help = 'Failed connecting through backwards-proxy, please restart the proxy configuration by disabling and enabling it.'
			// get proxy reachability
			axios.get(generateUrl('/apps/nextbox/forward/dyndns/test/proxy')).then((res) => {
				if (res.data.result === 'success') {
					this.internal.state = 'success'
					this.internal.icon = 'checkmark'
					this.internal.text = `Successfully tested reachability for: ${res.data.data.domain}`
				} else {
					this.internal.state = 'error'
					this.internal.icon = 'close'
					this.internal.text = `Failed reachability for: ${res.data.data.domain}`
				}
			}).catch((e) => {
				console.error(e)
				showError(t('nextbox', 'Connection Failed'))
			})
		}
	},
}
</script>


<style scoped>

.tag {
	margin-top: 8px;
	padding: 5px;
	border-radius: var(--border-radius);
	color: var(--color-primary-text);
	width: 90%;
	display: block;
	height: fit-content;
}

.tag-content {
	width: 100px; 
	overflow: hidden;
}

.tag-middle {
	width: 1px;
}

.tag-extra {
	/*width: 100px; */
	float: right; 
	text-align: right;
	padding-right: 10px;
}

.tag-action {
	padding-left: 8px;
	padding-right: 8px;
}

.tag-icon {
	opacity: 1;
	background-position-y: -1px;
	margin-right: 8px;
	margin-left: 4px;
	background-size: 16px;
	background-repeat: no-repeat;
	display: inline-block;
	vertical-align: middle !important;
}

.tag-help {
	margin-left: 10%;
	padding: 4px;
	width: 70%;
	background-color: #eee;
	display: block;
	border-left: solid 2px #bbb;
	border-right: solid 2px #bbb;
	border-bottom: solid 2px #bbb;
	border-radius: var(--border-radius);
}

</style>
