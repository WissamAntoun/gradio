<script lang="ts">
	import { createEventDispatcher } from "svelte";
	import { Upload, ModifyUpload } from "@gradio/upload";
	import type { FileData } from "@gradio/upload";
	import { Block, BlockLabel, Empty } from "@gradio/atoms";
	import { Chart } from "@gradio/chart";
	import UploadText from "../UploadText.svelte";

	import StatusTracker from "../StatusTracker/StatusTracker.svelte";
	import type { LoadingStatus } from "../StatusTracker/types";
	import { _ } from "svelte-i18n";

	import { Chart as ChartIcon } from "@gradio/icons";

	function format_value(val: StaticData) {
		return val.data.map((r) =>
			r.reduce((acc, next, i) => ({ ...acc, [val.headers[i]]: next }), {})
		);
	}

	const dispatch = createEventDispatcher<{
		change: undefined;
		clear: undefined;
	}>();

	interface StaticData {
		data: Array<Array<number>>;
		headers: Array<string>;
	}
	interface Data {
		data: Array<Array<number>> | string;
		headers?: Array<string>;
	}

	export let elem_id: string = "";
	export let elem_classes: Array<string> = [];
	export let visible: boolean = true;
	export let value: null | Data;
	export let y: Array<string>;
	export let x: string;
	export let mode: "static" | "dynamic";
	export let label: string;
	export let show_label: boolean;
	export let colors: Array<string>;

	export let loading_status: LoadingStatus;

	let _value: string | null;

	function data_uri_to_blob(data_uri: string) {
		const byte_str = atob(data_uri.split(",")[1]);
		const mime_str = data_uri.split(",")[0].split(":")[1].split(";")[0];

		const ab = new ArrayBuffer(byte_str.length);
		const ia = new Uint8Array(ab);

		for (let i = 0; i < byte_str.length; i++) {
			ia[i] = byte_str.charCodeAt(i);
		}

		return new Blob([ab], { type: mime_str });
	}

	function blob_to_string(blob: Blob) {
		const reader = new FileReader();

		reader.addEventListener("loadend", (e) => {
			//@ts-ignore
			_value = e.srcElement.result;
		});

		reader.readAsText(blob);
	}

	function dict_to_string(dict: Data) {
		if (dict.headers) _value = dict.headers.join(",");
		const data = dict.data as Array<Array<number>>;
		data.forEach((x: Array<unknown>) => {
			_value = _value + "\n";
			_value = _value + x.join(",");
		});
	}

	$: {
		if (value && value.data && typeof value.data === "string") {
			if (!value) _value = null;
			else blob_to_string(data_uri_to_blob(value.data));
		} else if (value && value.data && typeof value.data != "string") {
			if (!value) _value = null;
			dict_to_string(value);
		}
	}

	interface XRow {
		name: string;
		values: Array<number>;
	}

	interface YRow {
		name: string;
		values: Array<{ x: number; y: number }>;
	}

	function make_dict(x: XRow, y: Array<YRow>): Data {
		const headers = [];
		const data = [];

		headers.push(x.name);
		y.forEach(({ name }) => headers.push(name));

		for (let i = 0; i < x.values.length; i++) {
			let _data = [];
			_data.push(x.values[i]);
			y.forEach(({ values }) => _data.push(values[i].y));

			data.push(_data);
		}
		return { headers, data };
	}

	function handle_load(v: string | FileData | (string | FileData)[] | null) {
		value = { data: v as string };
		return v;
	}

	function handle_clear({ detail }: CustomEvent<FileData | null>) {
		value = null;
		dispatch("change");
		dispatch("clear");
	}

	$: _value = value == null ? null : _value;
	$: static_data =
		mode === "static" && value && format_value(value as StaticData);

	$: value, dispatch("change");
</script>

<Block
	{visible}
	variant={mode === "dynamic" && !_value ? "dashed" : "solid"}
	padding={false}
	{elem_id}
	{elem_classes}
>
	<BlockLabel {show_label} Icon={ChartIcon} label={label || "TimeSeries"} />
	<StatusTracker {...loading_status} />

	{#if mode === "static"}
		{#if static_data}
			<Chart value={static_data} {colors} />
		{:else}
			<Empty size="large" unpadded_box={true}>
				<ChartIcon />
			</Empty>
		{/if}
	{:else if _value}
		<div class="chart">
			<ModifyUpload on:clear={handle_clear} />
			<Chart
				value={_value}
				{y}
				{x}
				on:process={({ detail: { x, y } }) => (value = make_dict(x, y))}
				{colors}
			/>
		</div>
	{:else if value === undefined || value === null}
		<Upload
			filetype="text/csv"
			on:load={({ detail }) => handle_load(detail)}
			include_file_metadata={false}
		>
			<UploadText type="csv" />
		</Upload>
	{/if}
</Block>

<style>
	.chart {
		display: flex;
		display: relative;
		justify-content: center;
		align-items: center;
		background-color: var(--color-background-primary);
		width: var(--size-full);
		height: var(--size-64);
	}
</style>
