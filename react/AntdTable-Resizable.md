通过 react-resizable 重写 th。

重复计算渲染 columns，性能并不好。

```TypeScript
import { useState } from 'react';
import { Table, TableProps } from 'antd';
import { Resizable } from 'react-resizable';
import './index.css';
import React from 'react';

export interface ResizableTitleProps {
  onResize: () => void;
  width: number;
  [key: string]: any;
}

const ResizableTitle = ({ onResize, width, ...restProps }: ResizableTitleProps) => {
  if (!width) {
    return <th {...restProps} />;
  }

  return (
    <Resizable
      width={width}
      height={0}
      handle={
        <span
          className="react-resizable-handle"
          onClick={(e) => {
            e.stopPropagation();
          }}
        />
      }
      onResize={onResize}
      draggableOpts={{ enableUserSelectHack: false }}
    >
      <th {...restProps} style={{ ...restProps?.style, userSelect: 'none' }} />
    </Resizable>
  );
};

export const ResizableTable = ({ columns = [], ...props }: TableProps<any>) => {
  const [_, update] = useState({});

  const colsArray: any = columns.map((col, index) => {
    return {
      ...col,
      onHeaderCell: (column: any) => ({
        width: column.width,
        onResize: handleResize(index, col),
      }),
    };
  });

  const handleResize = (index: number, col: any) => {
    return (_: any, { size }: any) => {
      columns[index] = { ...col, width: size.width };
      update({})
    };
  };

  return (
    <Table
      components={{ header: { cell: ResizableTitle } }}
      columns={colsArray}
      {...props}
    />
  );
};

export default ResizableTable;
```

```CSS
.react-resizable-handle {
    position: absolute;
    right: -5px;
    bottom: 0;
    z-index: 1;
    width: 10px;
    height: 100%;
    cursor: col-resize;
}
```

