```TypeScript
import { message, PaginationProps } from 'antd';
import { useCallback, useEffect, useRef, useState } from 'react';

type Order = [string, 'desc' | 'asc'];
type Filter = [string, string, string];

export interface SearchParams {
  pageNum: number;
  pageSize: number;
  /* 高级搜索 */
  filters: Filter[];
  /* 模糊搜索值 */
  fuzzyKeyword?: string;
  /* 模糊搜索字段 */
  fuzzyFields?: string[];
  /* 排序 */
  orders?: Order[];
  selectedFields?: string[];
  modelCode?: string;
}

export interface Options<T> {
  /* 默认搜索参数 */
  defaultParams?: T;
  /* 手动请求 */
  manual?: boolean;
}

/* 响应 */
type Response<T> = {
  code: number;
  error: string;
  res: {
    total: number;
    records: T[];
    [key: string]: any;
  };
  trace: string;
};

type Api<T, K> = (params: T) => Promise<Response<K>>;

/**
 * 表格hook
 * @param api 请求接口
 * @param options 配置
 * @returns
 * @description 确保接口标准输入输出 (params: SearchParams) => Promise<Response<T>>
 */
export function useTable<T, K extends SearchParams = SearchParams>(
  api: Api<K, T>,
  options?: Options<Partial<K>>,
) {
  const [data, setData] = useState<T[]>([]);
  const [loading, setLoading] = useState(false);
  const [total, setTotal] = useState(0);

  /* 搜索参数 */
  const searchParams = useRef<K>({
    pageNum: 1,
    pageSize: 5,
    filters: [],
    /* 默认参数 */
    ...(options?.defaultParams as any),
  });

  /* 分页 */
  const pagination: PaginationProps = {
    current: searchParams.current.pageNum,
    pageSize: searchParams.current.pageSize,
    total,
    showTotal: (total) => `共${total}条`,
    showQuickJumper: true,
    showSizeChanger: true,
    onChange: (page: number, pageSize: number) => {
      searchParams.current.pageNum = page;
      searchParams.current.pageSize = pageSize;
      setLoading(true);
    },
  };

  const fetchApi = useCallback((params?: Partial<K>) => {
    searchParams.current = { ...searchParams.current, ...params };
    setLoading(true);
  }, []);

  useEffect(() => {
    console.log('loading :>> ', loading, searchParams.current);
    if (!loading) return;
    api(searchParams.current)
      .then((res) => {
        if (res.code !== 0) throw res.error;
        // setData(res.res.records);
        setTotal(res.res.total);
        setLoading(false);
        setData(res.res.records);
      })
      .catch((err) => {
        setLoading(false);
        message.error(err);
        console.error(err);
      });
  }, [loading]);

  useEffect(() => {
    if (options?.manual) return;
    fetchApi();
  }, []);

  useEffect(() => {
    if (!data.length && searchParams.current.pageNum > 1) {
      console.log('this page last :>> ');
      // @ts-ignore
      fetchApi({ pageNum: searchParams.current.pageNum - 1 });
    }
  }, [data.length]);

  return {
    data,
    loading,
    total,
    pagination,
    fetchApi,
  };
}

```

useEffect 钩子执行顺序和依赖项 set 的顺序有关。

这里 data 和 loading 这两个钩子，就和其更新的顺序挂钩。

