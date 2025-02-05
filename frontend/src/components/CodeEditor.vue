<template>
	<div
		class="editor"
		:style="{
			height: height,
		}">
		<span class="text-[10px] font-medium uppercase text-gray-600 dark:text-zinc-400">
			{{ label }}
		</span>
		<div ref="editor" class="border border-gray-200 dark:border-zinc-800" />
	</div>
</template>
<script setup lang="ts">
import { useDark } from "@vueuse/core";
import ace from "ace-builds";
import "ace-builds/src-noconflict/theme-chrome";
import "ace-builds/src-noconflict/theme-twilight";
import { PropType, onMounted, ref, watch } from "vue";
const isDark = useDark();

const props = defineProps({
	modelValue: {
		type: [Object, String, Array],
	},
	type: {
		type: String as PropType<"JSON" | "HTML" | "Python" | "JavaScript" | "CSS">,
		default: "JSON",
	},
	label: {
		type: String,
		default: "",
	},
	readonly: {
		type: Boolean,
		default: false,
	},
	height: {
		type: String,
		default: "200px",
	},
	showLineNumbers: {
		type: Boolean,
		default: false,
	},
	autofocus: {
		type: Boolean,
		default: true,
	},
});

const emit = defineEmits(["update:modelValue"]);

const editor = ref<HTMLElement | null>(null);
onMounted(() => {
	let initialValue = props.modelValue;
	const aceEditor = ace.edit(editor.value || "");
	aceEditor.setReadOnly(props.readonly);
	aceEditor.setOptions({
		fontSize: "12px",
		useWorker: false,
		showGutter: props.showLineNumbers,
	});
	if (props.type === "CSS") {
		import("ace-builds/src-noconflict/mode-css").then(() => {
			aceEditor.session.setMode("ace/mode/css");
		});
	} else if (props.type === "JavaScript") {
		import("ace-builds/src-noconflict/mode-javascript").then(() => {
			aceEditor.session.setMode("ace/mode/javascript");
		});
	} else if (props.type === "Python") {
		import("ace-builds/src-noconflict/mode-python").then(() => {
			aceEditor.session.setMode("ace/mode/python");
		});
	} else if (props.type === "JSON") {
		import("ace-builds/src-noconflict/mode-json").then(() => {
			aceEditor.session.setMode("ace/mode/json");
		});
		// check if initial value is valid json convert it to string
		try {
			initialValue = JSON.stringify(initialValue, null, 2);
		} catch (e) {
			// do nothing
		}
	} else {
		import("ace-builds/src-noconflict/mode-html").then(() => {
			aceEditor.session.setMode("ace/mode/html");
		});
	}
	aceEditor.on("blur", () => {
		try {
			let value = aceEditor.getValue();
			if (props.type === "JSON") {
				value = JSON.parse(value);
			}
			if (value === props.modelValue) return;
			emit("update:modelValue", value);
		} catch (e) {
			// do nothing
		}
	});
	watch(
		isDark,
		() => {
			aceEditor.setTheme(isDark.value ? "ace/theme/twilight" : "ace/theme/chrome");
		},
		{ immediate: true }
	);

	watch(
		() => props.modelValue,
		() => {
			let value = props.modelValue;
			if (props.type === "JSON") {
				value = JSON.stringify(value, null, 2);
			}
			aceEditor.setValue(value as string);
			aceEditor.clearSelection();
			props.autofocus && aceEditor.focus();
		},
		{ immediate: true }
	);
});
</script>
<style scoped>
.editor .ace_editor {
	height: 100%;
	width: 100%;
	border-radius: 5px;
	overscroll-behavior: none;
}
.editor :deep(.ace_scrollbar-h) {
	display: none;
}
</style>
