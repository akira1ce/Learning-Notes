```TypeScript
import { IntlShape } from 'react-intl';

/**
 * 重载 FormatMessage
 * @param intl intl 实例
 * @param localConfig 国际化配置
 * @returns formatMsg(id, defaultMessage)
 * @description 通过 localConfig 约束 id
 */
function overLoadFormatMessage<T>(intl: IntlShape, localConfig: T) {
  return function (id: keyof T & (string | number), defaultMessage: string) {
    return intl.formatMessage({ id, defaultMessage });
  };
}
export default overLoadFormatMessage;

/** use */
const formatMsg = overLoadFormatMessage(intl, userManage);
formatMsg('userManage.account', '账号'),
```

