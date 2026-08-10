
# Home & End Key Updates

```bash
mkdir -p ~/Library/KeyBindings
cat > ~/Library/KeyBindings/DefaultKeyBinding.dict << 'EOF'
{
  "\UF729"  = moveToBeginningOfLine:;                        /* Home */
  "\UF72B"  = moveToEndOfLine:;                              /* End */
  "$\UF729" = moveToBeginningOfLineAndModifySelection:;      /* Shift+Home */
  "$\UF72B" = moveToEndOfLineAndModifySelection:;            /* Shift+End */
  "^\UF729" = moveToBeginningOfDocument:;                    /* Ctrl+Home */
  "^\UF72B" = moveToEndOfDocument:;                          /* Ctrl+End */
  "$^\UF729" = moveToBeginningOfDocumentAndModifySelection:; /* Shift+Ctrl+Home */
  "$^\UF72B" = moveToEndOfDocumentAndModifySelection:;       /* Shift+Ctrl+End */
}
EOF
```
