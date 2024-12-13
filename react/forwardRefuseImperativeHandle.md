```TypeScript
export interface IRePasswordModalRef {
  handleChangeOpen: (open: boolean, userId: string) => void;
}

interface IRePasswordModalProps {
  onLoad: () => void;
}

const RePasswordModal = ({ onLoad }: IRePasswordModalProps, ref: ForwardedRef<IRePasswordModalRef>) => {
  useImperativeHandle(ref, () => ({
    handleChangeOpen,
  }));

  const handleChangeOpen = (open: boolean, userId: string = '') => {
    setOpen(open);
    form.setFieldValue('userId', userId);
  };

  return (<></>);
};

export default forwardRef(RePasswordModal);
```

