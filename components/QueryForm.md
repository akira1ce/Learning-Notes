```JavaScript
// QueryForm.js
import { useState } from 'react';

import { Form, Col, Row, Button } from 'antd';
import { UpOutlined, DownOutlined } from '@ant-design/icons';
import { getFields } from '../../utils/queryFilter';
import { btnStyle, labelCol } from '../../constants';

const QueryFrom = props => {
  const { form, columns, handleSearch, handleReset, hidenExpand = false } = props;
  const [expand, setExpand] = useState(false);

  const handleExpand = () => {
    setExpand(!expand);
  };

  return (
    <Form form={form} labelCol={labelCol}>
      <Row gutter={24}>
        {/* 查询条件 */}
        {getFields(columns, expand, hidenExpand)}
        <Col flex={1}>
          <Form.Item>
            <div style={btnStyle}>
              <Button type="link" hidden={hidenExpand} onClick={handleExpand}>
                {expand ? <UpOutlined /> : <DownOutlined />}
                {expand ? '收起' : '展开'}
              </Button>
              <Button onClick={handleReset}>重置</Button>
              <Button type="primary" onClick={handleSearch}>
                搜索
              </Button>
            </div>
          </Form.Item>
        </Col>
      </Row>
    </Form>
  );
};

export default QueryFrom;
```

```JavaScript
// queryFilter.js
const getPlaceHolder = (type, title) => {
  if (type === 'select') return `请选择${title}`;
  return `请输入${title}`;
};

export const returnElement = column => {
  const { title: label, dataIndex: name, valueType = 'text', valueEnum, rules, showTime = false } = column;

  let ValueComponent = null;

  const placeholder = getPlaceHolder(valueType, label);

  switch (valueType) {
    case 'text':
      ValueComponent = <Input placeholder={placeholder} allowClear/>;
      break;
    case 'select':
      ValueComponent = (
        <Select placeholder={placeholder} options={valueEnum} optionFilterProp="label" allowClear/>
      );
      break;
    case 'time':
        ValueComponent = (
          <FormDatePicker placeholder={placeholder} showTime={showTime}/>
        );
      break;
    // ...
    default:
      break;
  }
  return (
    <Col {...colProps}>
      <Form.Item name={name} label={label} rules={rules}>
        {ValueComponent}
      </Form.Item>
    </Col>
  );
};

export const getFields = (columns, expand, hidenExpand) => {
  if (!columns || !columns.length) return;

  columns = columns.filter(item => item.showSearch);

  const children = [];
  const len = columns.length;
  const count = hidenExpand || expand ? len : Math.min(5, len);

  for (let i = 0; i < count; i++) children.push(returnElement(columns[i]));

  return children;
};
```

```JavaScript
// constanst
export const colProps = { xs: 24, sm: 12, md: 12, lg: 12, xl: 8 };
export const labelCol = { sm: 10, md: 8, lg: 6, xl: 6, xxl: 4 };
export const btnStyle = { display: 'flex', justifyContent: 'end', gap: 10 };
```

