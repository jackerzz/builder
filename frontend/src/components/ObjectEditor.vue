<template>
	<div v-for="(value, key) in obj" :key="key" class="flex gap-2">
		<Input
			placeholder="Property"
			:modelValue="key"
			@update:modelValue="(val: string) => replaceKey(key, val)"
			class="rounded-md text-sm text-gray-800 dark:border-zinc-700 dark:bg-zinc-800 dark:text-zinc-200 dark:focus:bg-zinc-700" />
		<Input
			placeholder="Value"
			:modelValue="value"
			@update:modelValue="(val: string) => updateObjectValue(key, val)"
			class="rounded-md text-sm text-gray-800 dark:border-zinc-700 dark:bg-zinc-800 dark:text-zinc-200 dark:focus:bg-zinc-700" />
		<Button
			variant="outline"
			icon="x"
			class="p-2 dark:border-zinc-600 dark:bg-zinc-800 dark:text-gray-100 dark:outline-0 dark:hover:bg-zinc-700 dark:hover:text-gray-100"
			@click="deleteObjectKey(key as string)"></Button>
	</div>
	<Button
		variant="subtle"
		label="Add"
		class="dark:bg-zinc-800 dark:text-gray-100"
		@click="addObjectKey"></Button>
	<p
		class="rounded-sm bg-gray-100 p-2 text-2xs text-gray-800 dark:bg-zinc-800 dark:text-zinc-300"
		v-show="description">
		<span v-html="description"></span>
	</p>
</template>
<script setup lang="ts">
import { mapToObject, replaceMapKey } from "@/utils/helpers";

const props = defineProps<{
	obj: Record<string, string>;
	description?: string;
}>();

const emit = defineEmits({
	"update:obj": (obj: Record<string, string>) => true,
});

const addObjectKey = () => {
	const map = new Map(Object.entries(props.obj));
	map.set("", "");
	emit("update:obj", mapToObject(map));
};

const updateObjectValue = (key: string, value: string) => {
	const map = new Map(Object.entries(props.obj));
	map.set(key, value);
	emit("update:obj", mapToObject(map));
};

const replaceKey = (oldKey: string, newKey: string) => {
	const map = new Map(Object.entries(props.obj));
	emit("update:obj", mapToObject(replaceMapKey(map, oldKey, newKey)));
};

const deleteObjectKey = (key: string) => {
	const map = new Map(Object.entries(props.obj));
	map.delete(key);
	emit("update:obj", mapToObject(map));
};
</script>
