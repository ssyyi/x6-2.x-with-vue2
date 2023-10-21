<template>
  <div class="wrap">
    <!-- 左侧模块菜单 -->
    <div class="menu-bar">
      <!-- 模块列表 -->
      <div id="stencil"></div>
    </div>
    <!-- 按钮部分 -->
    <div class="canvas-card">
      <div>
        <button @click="exportPNG">导出PNG</button>
        <button @click="exportJSON">导出JSON</button>
        <button @click="importJSON">导入JSON</button>
      </div>
      <!-- 画布部分 -->
      <div id="container"></div>
    </div>
  </div>
</template>

<script>
import { Graph, Shape } from "@antv/x6";
import { Stencil } from "@antv/x6-plugin-stencil";
import { Transform } from "@antv/x6-plugin-transform";
import { Selection } from "@antv/x6-plugin-selection";
import { Snapline } from "@antv/x6-plugin-snapline";
import { Keyboard } from "@antv/x6-plugin-keyboard";
import { Clipboard } from "@antv/x6-plugin-clipboard";
import { History } from "@antv/x6-plugin-history";
import { Export } from "@antv/x6-plugin-export";
export default {
  data() {
    return {
      nodeList: [],
      graph: null,
      json: null,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    // 初始化 画布
    init() {
      const graph = new Graph({
        container: document.getElementById("container"),
        width: 1000,
        height: 800,
        grid: true,
        //滚轮
        mousewheel: {
          enabled: true,
          zoomAtMousePosition: true,
          modifiers: "ctrl",
          minScale: 0.5,
          maxScale: 3,
        },
        //连接线
        connecting: {
          router: "manhattan",
          connector: {
            name: "rounded",
            args: {
              radius: 8,
            },
          },
          anchor: "center",
          connectionPoint: "anchor",
          allowBlank: false,
          snap: {
            radius: 20,
          },
          createEdge() {
            return new Shape.Edge({
              attrs: {
                line: {
                  stroke: "#A2B1C3",
                  strokeWidth: 2,
                  targetMarker: {
                    name: "block",
                    width: 12,
                    height: 8,
                  },
                },
              },
              zIndex: 0,
            });
          },
          validateConnection({ targetMagnet }) {
            return !!targetMagnet;
          },
        },
        highlighting: {
          magnetAdsorbed: {
            name: "stroke",
            args: {
              attrs: {
                fill: "#5F95FF",
                stroke: "#5F95FF",
              },
            },
          },
        },
      });
      //使用插件
      graph
        .use(
          new Transform({
            resizing: true,
            rotating: true,
          })
        )
        .use(
          new Selection({
            rubberband: true,
            showNodeSelectionBox: true,
          })
        )
        .use(new Snapline())
        .use(new Keyboard())
        .use(new Clipboard())
        .use(new History())
        .use(new Export());
      //连接桩
      const ports = {
        groups: {
          top: {
            position: "top",
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: "#5F95FF",
                strokeWidth: 1,
                fill: "#fff",
                style: {
                  visibility: "hidden",
                },
              },
            },
          },
          right: {
            position: "right",
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: "#5F95FF",
                strokeWidth: 1,
                fill: "#fff",
                style: {
                  visibility: "hidden",
                },
              },
            },
          },
          bottom: {
            position: "bottom",
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: "#5F95FF",
                strokeWidth: 1,
                fill: "#fff",
                style: {
                  visibility: "hidden",
                },
              },
            },
          },
          left: {
            position: "left",
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: "#5F95FF",
                strokeWidth: 1,
                fill: "#fff",
                style: {
                  visibility: "hidden",
                },
              },
            },
          },
        },
        items: [
          {
            group: "top",
          },
          {
            group: "right",
          },
          {
            group: "bottom",
          },
          {
            group: "left",
          },
        ],
      };
      ///////////////////////////////
      const stencil = new Stencil({
        title: "流程图例",
        target: graph,
        stencilGraphWidth: 200,
        stencilGraphHeight: 180,
        collapsable: true,
        nodeMovable: true,
        groups: [
          {
            title: "基础流程图",
            name: "group1",
          },
        ],
        layoutOptions: {
          columns: 2,
          columnWidth: 80,
          rowHeight: 55,
        },
      });
      document.getElementById("stencil").appendChild(stencil.container);

      /////////////////////////////////////
      //注册节点
      Graph.registerNode(
        "custom-rect",
        {
          inherit: "rect",
          width: 66,
          height: 36,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: "#5F95FF",
              fill: "#EFF4FF",
            },
            text: {
              fontSize: 12,
              fill: "#262626",
            },
          },
          ports: { ...ports },
        },
        true
      );

      Graph.registerNode(
        "custom-polygon",
        {
          inherit: "polygon",
          width: 66,
          height: 36,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: "#5F95FF",
              fill: "#EFF4FF",
            },
            text: {
              fontSize: 12,
              fill: "#262626",
            },
          },
          ports: {
            ...ports,
            items: [
              {
                group: "top",
              },
              {
                group: "bottom",
              },
            ],
          },
        },
        true
      );

      Graph.registerNode(
        "custom-circle",
        {
          inherit: "circle",
          width: 45,
          height: 45,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: "#5F95FF",
              fill: "#EFF4FF",
            },
            text: {
              fontSize: 12,
              fill: "#262626",
            },
          },
          ports: { ...ports },
        },
        true
      );

      Graph.registerNode(
        "custom-image",
        {
          inherit: "rect",
          width: 52,
          height: 52,
          markup: [
            {
              tagName: "rect",
              selector: "body",
            },
            {
              tagName: "image",
            },
            {
              tagName: "text",
              selector: "label",
            },
          ],
          attrs: {
            body: {
              stroke: "#5F95FF",
              fill: "#5F95FF",
            },
            image: {
              width: 26,
              height: 26,
              refX: 13,
              refY: 16,
            },
            label: {
              refX: 3,
              refY: 2,
              textAnchor: "left",
              textVerticalAnchor: "top",
              fontSize: 12,
              fill: "#fff",
            },
          },
          ports: { ...ports },
        },
        true
      );
      //创建节点
      const r1 = graph.createNode({
        shape: "custom-rect",
        label: "开始",
        attrs: {
          body: {
            rx: 20,
            ry: 26,
          },
        },
      });
      const r2 = graph.createNode({
        shape: "custom-rect",
        label: "过程",
      });
      const r3 = graph.createNode({
        shape: "custom-rect",
        attrs: {
          body: {
            rx: 6,
            ry: 6,
          },
        },
        label: "可选过程",
      });
      const r4 = graph.createNode({
        shape: "custom-polygon",
        attrs: {
          body: {
            refPoints: "0,10 10,0 20,10 10,20",
          },
        },
        label: "判断",
      });
      const r5 = graph.createNode({
        shape: "custom-polygon",
        attrs: {
          body: {
            refPoints: "10,0 40,0 30,20 0,20",
          },
        },
        label: "数据",
      });
      const r6 = graph.createNode({
        shape: "custom-rect",
        label: "结束",
        attrs: {
          body: {
            rx: 20,
            ry: 26,
          },
        },
      });
      stencil.load([r1, r2, r3, r4, r5, r6], "group1");

      this.graph = graph;
      this.onMouse();
      this.KeyDown();
    },
    //键盘事件
    KeyDown() {
      this.graph.bindKey(["meta+c", "ctrl+c"], () => {
        const cells = this.graph.getSelectedCells();
        if (cells.length) {
          this.graph.copy(cells);
        }
        return false;
      });
      this.graph.bindKey(["meta+x", "ctrl+x"], () => {
        const cells = this.graph.getSelectedCells();
        if (cells.length) {
          this.graph.cut(cells);
        }
        return false;
      });
      this.graph.bindKey(["meta+v", "ctrl+v"], () => {
        if (!this.graph.isClipboardEmpty()) {
          const cells = this.graph.paste({ offset: 32 });
          this.graph.cleanSelection();
          this.graph.select(cells);
        }
        return false;
      });
      this.graph.bindKey(["meta+z", "ctrl+z"], () => {
        if (this.graph.canUndo()) {
          this.graph.undo();
        }
        return false;
      });
      this.graph.bindKey(["meta+shift+z", "ctrl+shift+z"], () => {
        if (this.graph.canRedo()) {
          this.graph.redo();
        }
        return false;
      });
      this.graph.bindKey(["meta+a", "ctrl+a"], () => {
        const nodes = this.graph.getNodes();
        if (nodes) {
          this.graph.select(nodes);
        }
      });
      this.graph.bindKey("backspace", () => {
        const cells = this.graph.getSelectedCells();
        if (cells.length) {
          this.graph.removeCells(cells);
        }
      });
      this.graph.bindKey(["ctrl+1", "meta+1"], () => {
        const zoom = this.graph.zoom();
        if (zoom < 1.5) {
          this.graph.zoom(0.1);
        }
      });
      this.graph.bindKey(["ctrl+2", "meta+2"], () => {
        const zoom = this.graph.zoom();
        if (zoom > 0.5) {
          this.graph.zoom(-0.1);
        }
      });
    },
    // 控制连接桩显示/隐藏
    showPorts(ports, show) {
      for (let i = 0, len = ports.length; i < len; i += 1) {
        ports[i].style.visibility = show ? "visible" : "hidden";
      }
    },
    onMouse() {
      this.graph.on("node:mouseenter", () => {
        const container = document.getElementById("container");
        const ports = container.querySelectorAll(".x6-port-body");
        this.showPorts(ports, true);
      });
      this.graph.on("node:mouseleave", () => {
        const container = document.getElementById("container");
        const ports = container.querySelectorAll(".x6-port-body");
        this.showPorts(ports, false);
      });
    },
    exportPNG() {
      this.graph.exportPNG();
    },
    exportJSON() {
      this.graph.exportJSNO();
      this.json = this.graph.toJSON();
      this.graph.clearCells();
    },
    importJSON() {
      this.graph.fromJSON(this.json);
    },
  },
};
</script>

<style lang="scss" scoped>
.wrap {
  margin: 0 200px;
  display: flex;
  width: 100vh;
  #stencil {
    width: 15vw;
    height: 100%;
    position: relative;
  }
  .canvas-card {
    margin: 0 10px;
    display: flex;
    flex-direction: column;
    button {
      height: 4vh;
      border: 1px solid #e8f4ff;
      border-radius: 5px;
      padding: 7px 15px;
      font-size: 12px;
      margin: 6px 12px;
      display: inline-block;
      line-height: 1;
      white-space: nowrap;
      box-sizing: border-box;
      outline: none;
      background: #e8f4ff;
      &:hover {
        background: #a3d3ff;
        cursor: pointer;
      }
    }
  }
}
</style>