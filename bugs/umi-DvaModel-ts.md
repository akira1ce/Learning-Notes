```TypeScript
/**
 * -- Reducers --
 * export type Reducer<S = any, A extends Action = AnyAction> = (prevState: S, action: A) => S;
 * 
 * -- Model --
 * export interface DvaModel<T, E = EffectsMapObject, R = ReducersMapObject<T>> {
 *  namespace: string,
 *  state?: T,
 *  reducers?: R,
 *  effects?: E,
 *  subscriptions?: SubscriptionsMapObject,
}
 */
import { getUserList } from "@/services/user";
import { DvaModel } from "@umijs/max";

interface IUser {
  _id?: string;
  account: string;
  avator: string;
  location: string;
  name: string;
  password: string;
  sex: number;
  role: string;
  status: number;
}

interface IState {
  userList: IUser[];
  name: string;
}

interface Action {
  type: string;
}

interface IUpdateUserListAction extends Action {
  payload: IState;
}

interface IReducers {
  updateUserList: Reducer<IState,IUpdateUserListAction>;
  ...
}

interface IEffects {
  getUserList: Effect;
  ...
}

export default {
  namespace: "user",
  state: {
    userList: [],
    name: "Akria",
  },
  effects: {
    *getUserList({ payload }: any, { call, put }: any) {
      const { data } = yield call(getUserList, payload);
      console.log(payload, data);
      yield put({ type: "updateUserList", payload: data });
    },
  },
  reducers: {
    updateUserList: (state, { payload }: any) => {
      return {
        ...state,
        ...payload,
      };
    },
    test(state, { payload }) {
      return {
        ...state,
        ...payload,
      };
    },
  },
} as DvaModel<IState,IEffects,IReducers>;

const userModel: IState = useSelector((state: any) => state.user)

const dispatch = useDispatch()
dispatch<IUpdateUserListAction>({ type: 'xx', payload: {} })
```

