```TypeScript
import React, {
  forwardRef,
  memo,
  useCallback,
  useEffect,
  useImperativeHandle,
  useMemo,
  useRef,
  useState,
} from 'react';
import { message, Select, SelectProps } from 'antd';
// @ts-ignore
import { cloneDeep, debounce, get, isEqual, uniqBy } from 'lodash';
// @ts-ignore
import { request } from 'cn-lib';

type Filter = [string, string, string];
type Order = [string, 'desc' | 'asc'];
type OmitSelectProps = 'options' | 'loading' | 'onSelect' | 'labelInValue';

export interface SearchParams {
  pageNum?: number;
  pageSize?: number;
  fuzzyKeyword?: string;
  fuzzyFields?: string[];
  filters?: Filter[];
  orders?: Order[];
  modelCode?: string;
}

interface Res {
  records: any[];
  total: number;
}

interface Response {
  res: Res;
  error: string | null;
  code: number;
  trace: string | null;
}

export interface RemoteSelectProps extends Omit<SelectProps, OmitSelectProps> {
  /* 请求 */
  api: (params?: SearchParams) => Promise<Response>;
  /* 下拉初始值 */
  defaultOptions?: SelectProps['options'];
  /* 搜索参数 */
  searchParams?: SearchParams;
  /* value 字段 支持 xxx.xxx 嵌套类型 */
  valueField?: string;
  /* label 字段 支持 xxx.xxx 嵌套类型 */
  labelField?: string;
}

export interface RemoteSelectRef {}

const DEFAULT_SEARCHPARAMS: SearchParams = {
  pageNum: 1,
  pageSize: 10,
  fuzzyFields: ['name'],
};

/**
 * 远程下拉组件
 * 由于下拉数据需要走请求，导致回显存在异常情况，默认采用 labelInValue 模式
 * 除 'options' | 'loading' | 'onSelect' | 'labelInValue' 字段，支持原生 antd Select 所有属性
 */
const RemoteSelect: React.ForwardRefRenderFunction<RemoteSelectRef, RemoteSelectProps> = (
  props,
  ref,
) => {
  const {
    api,
    defaultOptions = [],
    searchParams,
    labelField = 'name',
    valueField = 'id',
    ...selectProps
  } = props;

  const { value, showSearch } = selectProps;

  const [loading, setLoading] = useState(false);
  const [options, setOptions] = useState<SelectProps['options']>(defaultOptions);
  const [extraOptions, setExtraOptions] = useState<SelectProps['options']>([]);

  const isManualSelect = useRef(false);
  const preSearchParams = useRef<SearchParams>();

  /* options 去重 */
  const optionsUniq = useMemo(() => {
    if (!extraOptions.length) return options;
    return uniqBy([...extraOptions, ...options], 'value');
  }, [options, extraOptions]);

  // console.log('extraOptions :>> ', extraOptions);
  // console.log('optionsUniq :>> ', optionsUniq);

  /* api-params */
  const params = useRef({ ...DEFAULT_SEARCHPARAMS });

  const handleSearch = useCallback(
    debounce((val: string) => {
      params.current.fuzzyKeyword = val;
      setLoading(true);
    }, 300),
    [],
  );

  useImperativeHandle(ref, () => ({}));

  useEffect(() => {
    /* 深比较是否相等 */
    if (isEqual(preSearchParams.current, searchParams)) return;
    preSearchParams.current = cloneDeep(searchParams);

    params.current = {
      ...params.current,
      ...searchParams,
    };
    setLoading(true);
  }, [searchParams]);

  useEffect(() => {
    // @ts-ignore
    if (!loading) return;
    let lock = false;
    try {
      api(params.current).then((ress) => {
        if (lock) return;
        if (ress.code !== 0) throw new Error(ress.error || '未知错误');

        const _options = ress.res?.records?.map((item, index: number) => ({
          label: get(item, labelField)?.toString() || `${labelField}字段不存在`,
          value: get(item, valueField)?.toString() || `${valueField}字段不存在`,
        }));

        setOptions(_options);
        setLoading(false);
      });
    } catch (err: any) {
      message.error(err.message || err.toString());
      setLoading(false);
    }
    return () => {
      lock = true;
    };
  }, [loading]);

  useEffect(() => {
    /* 手动选择 */
    if (isManualSelect.current) {
      isManualSelect.current = false;
      setExtraOptions([]);
      return;
    }

    /* 空数据 */
    if (!value || !value?.value) return;

    /* 额外 Option */
    setExtraOptions([{ value: value.value, label: value.label }]);
  }, [value?.value]);

  return (
    <Select
      {...selectProps}
      showSearch={showSearch}
      filterOption={false}
      loading={loading}
      options={optionsUniq}
      onSearch={showSearch ? handleSearch : undefined}
      onSelect={() => (isManualSelect.current = true)}
      labelInValue
    />
  );
};

export default memo(forwardRef(RemoteSelect));

```



