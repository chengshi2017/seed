<script>
import SeedTable from "#/components/SeedTable/Index.vue";
import SeedForm from "./SeedForm.vue";
import optionsHelper from "#/lib/options";
import { Message } from "element-ui";

const injectErrorCatch = function(promise) {
  promise.catch(e => {
    Message.error(e.message);
  });
};

export default {
  components: {
    SeedTable,
    SeedForm
  },
  name: "SeedTableContainer",
  props: {
    seeds: {
      type: Array,
      required: true
    },
    addItem: {
      type: Function
    },
    updateItem: {
      type: Function
    },
    deleteItem: {
      type: Function
    }
  },
  computed: {
    tableSeeds({
      seeds,
      $parent,
      updateItem,
      deleteItem,
      openForm,
      deleteRow
    }) {
      const tableSeeds = optionsHelper(seeds, "table");

      // 判断是否需要增加工具列
      const scopeControl = $parent.$scopedSlots.control;
      if (scopeControl || updateItem || deleteItem) {
        tableSeeds.push({
          key: "_control",
          label: "操作",
          render(h, props) {
            const { row, ...scope } = props;
            return (
              <div class="control-column">
                {scopeControl && scopeControl({ row, ...scope })}
                {updateItem && (
                  <ElButton
                    type="warning"
                    size="small"
                    onClick={() => openForm(row)}
                  >
                    编辑
                  </ElButton>
                )}
                {deleteItem && (
                  <ElPopconfirm
                    confirmButtonText="确定"
                    cancelButtonText="取消"
                    title="您确定删除该记录么？"
                    onOnConfirm={() => deleteRow(row)}
                  >
                    <ElButton type="danger" size="small" slot="reference">
                      删除
                    </ElButton>
                  </ElPopconfirm>
                )}
              </div>
            );
          }
        });
      }

      return tableSeeds;
    },
    // 给table用的渲染data
    tableVnodeData({ $attrs, $listeners, $parent, openForm, addItem, $slots }) {
      const { batchList } = $parent.$scopedSlots;
      const { tools } = $parent.$slots;
      return {
        props: $attrs,
        on: $listeners,
        scopedSlots: {
          batchList,
          default: () => {
            return (
              <div>
                {
                  // slot的工具栏
                  (tools, $slots.tools)
                }
                {// 添加按钮
                addItem && (
                  <ElButton
                    size="small"
                    type="primary"
                    onClick={() => openForm()}
                  >
                    添加
                  </ElButton>
                )}
              </div>
            );
          }
        }
      };
    },
    // 给form用的基础渲染data
    formVnodeData({ $attrs, $listeners }) {
      return {
        props: $attrs,
        on: $listeners
      };
    }
  },
  methods: {
    openForm(defaultForm) {
      const form = this.$refs.form;
      if (form) {
        form.open(defaultForm);
      } else {
        throw new Error("form生成失败");
      }
    },
    refresh() {
      this.$parent.refresh();
    },
    submit({ form, isAdd } = {}) {
      if (form) {
        const promise = this.$props[isAdd ? "addItem" : "updateItem"](form);
        if (promise instanceof Promise) {
          injectErrorCatch(promise.then(() => this.refresh()));
        } else {
          this.$nextTick(() => {
            setTimeout(() => {
              this.refresh();
            }, 500);
          });
        }
      }
    },
    deleteRow(row) {
      const promise = this.deleteItem(row);
      if (promise instanceof Promise) {
        injectErrorCatch(promise.then(() => this.refresh()));
      } else {
        this.$nextTick(() => {
          setTimeout(() => {
            this.refresh();
          }, 500);
        });
      }
    }
  },
  // 改用render函数，保证slot的传递
  render() {
    const {
      tableSeeds,
      addItem,
      updateItem,
      seeds,
      formVnodeData,
      tableVnodeData,
      submit
    } = this;

    return (
      <div class="table-container">
        <SeedTable seeds={tableSeeds} {...tableVnodeData} />
        {(addItem || updateItem) && (
          <SeedForm
            onSubmit={submit}
            seeds={seeds}
            ref="form"
            {...formVnodeData}
          />
        )}
      </div>
    );
  }
};
</script>

<style lang="stylus" scoped>
.table-container
  margin 16px 0
  >>> .control-column
    display inline-flex
    justify-content space-between
    min-width 120px
</style>
