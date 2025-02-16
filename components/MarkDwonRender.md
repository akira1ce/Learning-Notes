# markdown-render

react-markdown 自定义渲染 markdown 数据。

[react-markdown](https://github.com/remarkjs/react-markdown)

```tsx
import ReactMarkdown from 'react-markdown';
import { CodeProps } from 'react-markdown/lib/ast-to-react';
import remarkGfm from 'remark-gfm';

export interface MarkDwonRenderProps {
  value: string;
}

/* 代码块 */
function Code(props: CodeProps) {
  return (
    <div style={{ backgroundColor: '#F6F8FA', padding: 10 }}>
      <code {...props}></code>
    </div>
  );
}

/* 行代码块 */
function CodeInline(props: CodeProps) {
  return (
    <span style={{ backgroundColor: '#F6F8FA', padding: 2 }}>
      <code {...props}></code>
    </span>
  );
}

/* 表格 */
function genTable() {
  return {
    table: (props: any) => (
      <table style={{ width: '100%', borderCollapse: 'collapse', margin: '20px 0' }}>
        {props.children}
      </table>
    ),
    th: (props: any) => (
      <th
        style={{
          padding: '10px',
          backgroundColor: '#f2f2f2',
          border: '1px solid #ddd',
          textAlign: 'left',
        }}
      >
        {props.children}
      </th>
    ),
    td: (props: any) => (
      <td style={{ padding: '8px', border: '1px solid #ddd', textAlign: 'left' }}>
        {props.children}
      </td>
    ),
    tr: (props: any) => <tr style={{ backgroundColor: '#fff' }}> {props.children}</tr>,
  };
}

function MarkDwonRender(props: MarkDwonRenderProps) {
  const { value } = props;

  return (
    <ReactMarkdown
      remarkPlugins={[remarkGfm]}
      components={{
        code(props) {
          const { node, ...rest } = props;
          const { inline } = rest;
          if (inline) return <CodeInline {...props} />;
          return <Code {...props} />;
        },
        ...genTable(),
      }}
    >
      {value}
    </ReactMarkdown>
  );
}

export default MarkDwonRender;
```