<script setup lang="ts">
import { computed, ref } from "vue";
import { ElMessage } from "element-plus";
import * as XLSX from "xlsx";

type JsonRow = Record<string, string | number | boolean | null>;

const fileName = ref<string>("");
const workbookRef = ref<XLSX.WorkBook | null>(null);
const sheetName = ref<string>("");
const jsonData = ref<JsonRow[]>([]);

const sheetOptions = computed(() => {
	if (!workbookRef.value) {
		return [];
	}
	return workbookRef.value.SheetNames.map((name) => ({
		label: name,
		value: name,
	}));
});

const jsonText = computed(() => {
	if (!jsonData.value.length) {
		return "";
	}
	return JSON.stringify(jsonData.value, null, 2);
});

const tableColumns = computed(() => {
	const keys = new Set<string>();
	jsonData.value.forEach((row) => {
		Object.keys(row).forEach((key) => keys.add(key));
	});
	return Array.from(keys);
});

const resetInputValue = (event: Event) => {
	const target = event.target as HTMLInputElement | null;
	if (target) {
		target.value = "";
	}
};

const updateJsonBySheet = () => {
	if (!workbookRef.value || !sheetName.value) {
		jsonData.value = [];
		return;
	}
	const worksheet = workbookRef.value.Sheets[sheetName.value];
	if (!worksheet) {
		jsonData.value = [];
		return;
	}
	jsonData.value = XLSX.utils.sheet_to_json<JsonRow>(worksheet, {
		defval: "",
	});
};

const handleFileChange = async (event: Event) => {
	const target = event.target as HTMLInputElement | null;
	const file = target?.files?.[0];
	if (!file) {
		return;
	}
	if (!/\.(xlsx|xls)$/i.test(file.name)) {
		ElMessage.error("仅支持 .xlsx 或 .xls 文件");
		resetInputValue(event);
		return;
	}
	try {
		const buffer = await file.arrayBuffer();
		const workbook = XLSX.read(buffer, { type: "array" });
		if (!workbook.SheetNames.length) {
			ElMessage.warning("该文件未包含有效工作表");
			resetInputValue(event);
			return;
		}
		workbookRef.value = workbook;
		fileName.value = file.name;
		sheetName.value = workbook.SheetNames[0];
		updateJsonBySheet();
		resetInputValue(event);
	} catch (error) {
		console.log(error);
		ElMessage.error("读取失败，请检查文件内容");
		resetInputValue(event);
	}
};

const downloadJsonFile = () => {
	if (!jsonData.value.length) {
		ElMessage.warning("暂无可导出的数据");
		return;
	}
	const data = JSON.stringify(jsonData.value, null, 2);
	const blob = new Blob([data], { type: "application/json;charset=utf-8" });
	const url = URL.createObjectURL(blob);
	const link = document.createElement("a");
	const safeName = fileName.value ? fileName.value.replace(/\.[^/.]+$/, "") : "excel-data";
	link.href = url;
	link.download = `${safeName}.json`;
	link.click();
	URL.revokeObjectURL(url);
};

const clearData = () => {
	fileName.value = "";
	workbookRef.value = null;
	sheetName.value = "";
	jsonData.value = [];
};
</script>

<template>
	<div class="excel-tool-page">
		<el-card shadow="never">
			<template #header>
				<div class="excel-tool-header">
					<div class="title">Excel 转 JSON 工具</div>
					<div class="actions">
						<label class="upload-btn">
							<el-button type="primary">选择 Excel 文件</el-button>
							<input type="file" accept=".xlsx,.xls" @change="handleFileChange" />
						</label>
						<el-button :disabled="!jsonData.length" @click="downloadJsonFile">导出 JSON 文件</el-button>
						<el-button :disabled="!jsonData.length && !fileName" @click="clearData">清空</el-button>
					</div>
				</div>
			</template>

			<el-alert
				type="info"
				show-icon
				:closable="false"
				title="支持 .xlsx / .xls 文件。默认读取第一个工作表，可切换工作表后重新生成 JSON。"
			/>

			<el-divider />

			<el-row :gutter="16">
				<el-col :xs="24" :sm="24" :md="8">
					<el-descriptions title="文件信息" :column="1" border>
						<el-descriptions-item label="文件名">
							{{ fileName || "未选择" }}
						</el-descriptions-item>
						<el-descriptions-item label="工作表">
							<el-select
								v-model="sheetName"
								placeholder="请选择工作表"
								:disabled="!sheetOptions.length"
								style="width: 100%"
								@change="updateJsonBySheet"
							>
								<el-option v-for="item in sheetOptions" :key="item.value" :label="item.label" :value="item.value" />
							</el-select>
						</el-descriptions-item>
						<el-descriptions-item label="行数">
							{{ jsonData.length }}
						</el-descriptions-item>
					</el-descriptions>
				</el-col>

				<el-col :xs="24" :sm="24" :md="16">
					<el-input
						:model-value="jsonText"
						type="textarea"
						:rows="10"
						readonly
						placeholder="读取 Excel 后将展示 JSON 内容"
					/>
				</el-col>
			</el-row>

			<el-divider />

			<el-table v-if="jsonData.length" :data="jsonData" height="420" border>
				<el-table-column
					v-for="col in tableColumns"
					:key="col"
					:prop="col"
					:label="col"
					show-overflow-tooltip
				/>
			</el-table>
			<el-empty v-else description="暂无数据，请先选择 Excel 文件" />
		</el-card>
	</div>
</template>

<style scoped lang="scss">
.excel-tool-page {
	padding: 16px;
}

.excel-tool-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 16px;

	.title {
		font-size: 16px;
		font-weight: 600;
	}

	.actions {
		display: flex;
		gap: 12px;
		align-items: center;
	}
}

.upload-btn {
	position: relative;
	display: inline-flex;
	align-items: center;

	input[type="file"] {
		position: absolute;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		opacity: 0;
		cursor: pointer;
	}
}
</style>
