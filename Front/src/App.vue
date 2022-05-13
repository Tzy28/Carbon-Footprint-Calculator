<template>
  <div>
    <el-container>
      <el-header>Medical Product Carbon Footprint Calculator <i class="el-icon-warning zs" @click="showTip=true"></i> </el-header>
      <el-main>
        <el-row>
          <el-button type="primary" plain @click="dialogTableVisible=true"> Add </el-button>
          <el-button type="success" plain @click="sumAll"> Calculate </el-button>
          <!-- <el-button type="primary" plain @click="clearAll"> Clear All </el-button> -->
        </el-row>
        <el-table :data="productData" border style="width: 100%">
          <el-table-column label="Product Name" align="center">
            <template slot-scope="scope">
              <span style="center">{{scope.row.type}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Quantity" align="center">
            <template slot-scope="scope">
              <span style="center">{{scope.row.quantity}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Weight" align="center">
            <template slot-scope="scope">
              <span style="center">{{scope.row.weight}}</span>
            </template>
          </el-table-column>
          <el-table-column label="Operation" align="center">
            <template slot-scope="scope">
              <el-button size="mini" @click="handleEdit(scope.$index,scope.row)">Edit</el-button>
              <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">Delete</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-main>
      <el-footer style="height:120px">
        <div>Result: </div>
        <div>Your CO₂ Emission: {{sum.toFixed(2)}} g</div>
        <div>Equivalent Vehicle Driving Distance: {{kilometer.toFixed(2)}} km </div>
      </el-footer>
    </el-container>

    <el-dialog :title="title" :visible.sync="dialogTableVisible" width="500px" @close="close">
      <el-form ref="form" :rules="rules" :model="form" label-position="top">
        <el-form-item label="Please select your product" prop="type">
          <el-cascader v-model="form.showType" :options="selectList" @change="handleChange" style="width:300px" aria-placeholder="Please Select"></el-cascader>
        </el-form-item>
        <el-form-item label="Quantity" prop="quantity">
          <el-input v-model="form.quantity" style="width:300px" type="number"></el-input>
        </el-form-item>
        <el-form-item label="Weight (Gram)" v-if="form.children" prop="weight">
          <el-input v-model="form.weight" style="width:300px" type="number"></el-input>
        </el-form-item>
        <el-form-item label="" class="mt50">
          <div class="f-r f-a-c f-j-c">
            <el-button type="primary" @click="submit">{{title=='Add'?'Add':'Edit'}}</el-button>
            <el-button type="danger" @click="close">Cancel</el-button>
          </div>
        </el-form-item>
      </el-form>
    </el-dialog>

    <el-dialog title="Guide" :visible.sync="showTip" width="500px">
      <p><span class="p-r" style="top:-2px">1.</span> Click "Add" to add product.</p>
      <p><span class="p-r" style="top:-2px">2.</span> Select the product and fill the details.</p>
      <p><span class="p-r" style="top:-2px">3.</span> You can edit or remove items in the list.</p>
      <p><span class="p-r" style="top:-2px">4.</span> Click "Calculate" to get the result on selected items.</p>
    </el-dialog>
  </div>
</template>

<script>
import "@/assets/public.css";

export default {
  data() {
    return {
      dialogTableVisible: false,
      title: "Add",
      showTip: false,
      sum: 0,
      kilometer: 0,
      value: "",
      rules: {
        type: [
          {
            validator: (rule, value, callback) => {
              if (value === "" || value === undefined) {
                callback(new Error("Please enter data"));
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
        quantity: [
          {
            validator: (rule, value, callback) => {
              if (value === "" || value === undefined) {
                callback(new Error("Please enter data"));
              } else if (!/(^[1-9]\d*$)/.test(value) || value == 0) {
                callback(new Error("Invalid data"));
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
        weight: [
          {
            validator: (rule, value, callback) => {
              if (value === "" || value === undefined) {
                callback(new Error("Please enter data"));
              } else if (value.indexOf("-") > -1 || value == 0) {
                callback(new Error("Invalid data"));
              } else {
                callback();
              }
            },
            trigger: "blur",
          },
        ],
      },
      form: {
        showType: "",
        type: "",
        quantity: "",
        weight: "",
        children: false,
      },
      selectList: [
        {
          value: "Exam Glove",
          label: "Exam Glove",
          children: [
            {
              value: "Non-Sterile",
              label: "Non-Sterile",
              children: [
                {
                  value: "Nitrile",
                  label: "Nitrile",
                },
                {
                  value: "Latex",
                  label: "Latex",
                },
              ],
            },
            {
              value: "Sterile",
              label: "Sterile",
              children: [
                {
                  value: "Nitrile",
                  label: "Nitrile",
                },
                {
                  value: "Latex",
                  label: "Latex",
                },
              ],
            },
          ],
        },
        {
          value: "Gown",
          label: "Gown",
        },
        {
          value: "Face Shield",
          label: "Face Shield",
        },
        {
          value: "Apron",
          label: "Apron",
        },
        {
          value: "Surgical Mask",
          label: "Surgical Mask",
        },
        {
          value: "Hand Sanitizer",
          label: "Hand Sanitizer",
        },
        {
          value: "Hand Washing",
          label: "Hand Washing",
        },
        {
          value: "N95 Mask",
          label: "N95 Mask",
        },
        {
          value: "Surgical Mask",
          label: "Surgical Mask",
        }
      ],
      productData: [],
    };
  },
  created() {
    this.productData = JSON.parse(localStorage.getItem("productData")) || [];
  },
  methods: {
    close() {
      this.dialogTableVisible = false;
      this.form = {
        type: "",
        quantity: "",
        weight: "",
        children: false,
      };
    },

    submit() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (this.title == "Add") {
            this.productData.push({
              type: this.form.type,
              showType: this.form.showType,
              quantity: this.form.quantity,
              weight: this.form.weight,
              children: this.form.children,
            });
          } else {
            this.$set(this.productData, this.form.index, {
              type: this.form.type,
              showType: this.form.showType,
              quantity: this.form.quantity,
              weight: this.form.weight,
              children: this.form.children,
            });
          }
          localStorage.setItem("productData", JSON.stringify(this.productData));
          this.close();
        } else {
          return false;
        }
      });
    },
    handleChange(value) {
      if (!value) {
        return;
      }
      if (value.length == 3) {
        this.form.type = value.join("-");
        this.form.children = true;
      } else {
        this.form.type = value[0];
        this.form.children = false;
      }
    },
    handleEdit(index, row) {
      this.form = JSON.parse(JSON.stringify({ ...row, index: index }));
      this.title = "Edit";
      this.dialogTableVisible = true;
    },
    handleDelete(index, row) {
      this.productData.splice(index, 1);
      localStorage.setItem("productData", JSON.stringify(this.productData));
    },

    sumOne(type, weight, quantity) {
      let sum = 0;
      if (type == "Exam Glove-Non-Sterile-Nitrile") {
        sum = (2.68 * Number(weight) + 17.33) * Number(quantity);
      } else if (type == "Exam Glove-Non-Sterile-Latex") {
        sum = (1.6 * Number(weight) + 32) * Number(quantity);
      } else if (type == "Exam Glove-Sterile-Nitrile") {
        sum = (2.68 * Number(weight) + 17.33 + 1.5) * Number(quantity);
      } else if (type == "Exam Glove-Sterile-Latex") {
        sum = (1.6 * Number(weight) + 32 + 1.5) * Number(quantity);
      } else if (type == "Gown") {
        sum = 905 * Number(quantity);
      } else if (type == "Face Shield") {
        sum = 231 * Number(quantity);
      } else if (type == "Apron") {
        sum = 65 * Number(quantity);
      } else if (type == "Hand Sanitizer") {
        sum = 0.9 * Number(quantity);
      } else if (type == "Hand Washing") {
        sum = 9 * Number(quantity);
      } else if (type == "N95 Mask"){
        sum = 50 * Number(quantity);
      } else if (type == "Surgical Mask"){
        sum = 59 * Number(quantity);
      }
      return Number(sum.toFixed(2));
    },

    sumAll() {
      let sum = 0;
      let kilometer = 0;
      this.productData.forEach((item) => {
        sum += this.sumOne(item.type, item.weight, item.quantity);
      });
      this.sum = Number(sum);
      this.kilometer = (sum/404)*1.6;
    },
  },
};
</script>

<style>
.el-header,
.el-footer {
  background-color: #87c26a;
  color: #333;
  text-align: center;
  line-height: 40px;
}

.el-main {
  background-color: #ffffff;
  color: #333;
  text-align: center;
  line-height: 160px;
}

body > .el-container {
  margin-bottom: 40px;
}

.el-table {
  align-items: center;
  justify-content: center;
}
</style>
